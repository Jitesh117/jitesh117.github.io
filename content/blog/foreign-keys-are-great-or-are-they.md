+++
title = "Foreign Keys Are Great. Or are they?"
date = 2025-08-31T10:42:21+05:30
draft = false
showToc = true
cover.image = '/images/FK.jpg'
tags = ['tech', 'SQL']
+++

Last week I was knee-deep in building the backend for my project. I had just wrapped up the user endpoints and was ready to tackle posts. That's when I made a classic mistake - I jumped straight into coding without doing any proper database modeling or planning.

Here's what I initially threw together:

```go
type User struct {
	gorm.Model
	UserTag        string `json:"user_tag"         gorm:"unique;not null;size:50"`
	Name           string `json:"name"`
	Password       string `json:"-"                gorm:"not null"`
	Email          string `json:"email"            gorm:"unique;not null"`
	ProfilePicture string `json:"profile_picture"`
	GithubUserName string `json:"github_user_name"`
}

type Rice struct {
	gorm.Model
	Title       string `json:"title"`
	Description string `json:"description"`
	RepoLink    string `json:"repo_link"`
	CoverImage  string `json:"cover_image"`
	UserTag     string `json:"user_tag"    gorm:"not null;size:50"`
}
```

The problem was obvious - even though my `Rice` struct had a `UserTag` field, there was zero referential integrity. It was basically just a string floating around with no guarantees.

I saw two ways to fix this mess:

1. Properly refactor everything to use foreign key constraints
2. Hack together a quick fix with transactions

So I did what any reasonable developer would do - I went down a rabbit hole researching the pros and cons. Most of what I found came from [this HackerNews thread](https://news.ycombinator.com/item?id=32731916), and honestly, it scared me off foreign keys:

> Foreign keys are notorious during Data Migrations.

> ORMs make it slow while working with FKs.

> Foreign Key constraints slow down Insert and Update statements.

Yeah, I know these were pretty scattered opinions, but they somehow convinced me that foreign keys were the devil. So I went with the transaction approach.

I tried getting Claude to help with the refactor, but no matter how I phrased it, it just couldn't get the job done. So I ended up writing this transaction to handle user tag updates:

```go
if userTagChanged {
		err := h.DB.Transaction(func(tx *gorm.DB) error {
			if err := tx.Model(&user).Updates(user).Error; err != nil {
				return err
			}

			if err := tx.Model(&models.Rice{}).Where("user_tag = ?", originalUserTag).Update("user_tag", user.UserTag).Error; err != nil {
				return err
			}

			return nil
		})
		if err != nil {
			c.JSON(
				http.StatusInternalServerError,
				gin.H{"error": "Failed to update user and rice records: " + err.Error()},
			)
			return
		}
	} else {
		if err := h.DB.Model(&user).Updates(user).Error; err != nil {
			c.JSON(http.StatusInternalServerError, gin.H{"error": "Failed to update user"})
			return
		}
	}
```

But something felt off about this whole approach. My gut was telling me I was overcomplicating things.

So I posted about it in [The Engineer Org's Discord server](https://discord.gg/A4xjNNRqJv). Pretty much everyone told me to just use foreign keys - they exist for a reason. But what really hit home was this comment from [svs](https://x.com/_svs_):

> Always 'push the problem down'. The db is a very powerful piece of tech and you should be telling it the minimal amount of things and depend on it to do the heavy lifting for you.

That was it. I was convinced. Time to embrace foreign keys properly.

Here's what the models look like now:

```go
type User struct {
	ID             uuid.UUID      `json:"id"               gorm:"type:uuid;primary_key;default:uuid_generate_v4()"`
	CreatedAt      time.Time      `json:"created_at"`
	UpdatedAt      time.Time      `json:"updated_at"`
	DeletedAt      gorm.DeletedAt `json:"deleted_at"       gorm:"index"`
	UserTag        string         `json:"user_tag"         gorm:"unique;not null;size:50"`
	Name           string         `json:"name"`
	Password       string         `json:"-"                gorm:"not null"`
	Email          string         `json:"email"            gorm:"unique;not null"`
	ProfilePicture string         `json:"profile_picture"`
	GithubUserName string         `json:"github_user_name"`
	Rices          []Rice         `json:"-"                gorm:"foreignKey:UserID"`
}

type Rice struct {
	ID          uuid.UUID      `json:"id"          gorm:"type:uuid;primary_key;default:uuid_generate_v4()"`
	CreatedAt   time.Time      `json:"created_at"`
	UpdatedAt   time.Time      `json:"updated_at"`
	DeletedAt   gorm.DeletedAt `json:"deleted_at"  gorm:"index"`
	Title       string         `json:"title"       gorm:"not null;check:title <> ''"`
	Description string         `json:"description"`
	RepoLink    string         `json:"repo_link"`
	CoverImage  string         `json:"cover_image"`
	UserID      uuid.UUID      `json:"user_id"     gorm:"not null;size:50;type:uuid"`
	User        User           `json:"-"           gorm:"constraint:OnUpdate:CASCADE,foreignKey:UserID;references:ID"`
}
```

The difference is night and day. Everything's cleaner, the database handles referential integrity automatically, and I can trust that my data relationships are solid.

Want to know the kicker? This entire refactor added less than 5 lines of new code. Most of the work was actually _deleting_ all the redundant transaction logic I'd written to work around not having proper foreign keys.

Sometimes the right solution really is the simplest one.

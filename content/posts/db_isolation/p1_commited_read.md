---
title: "Transactions Isolation #2: Phantom Menace"
description: "The second post in a series of posts about database transactions and isolation levels. This one is about READ_COMMITED and phantom rows."
date: 2022-05-20T11:13:37+02:00
categories: 
- Development
series:
- 'Transaction Isolation'
tags:
- Database
- Go
- Concurrency
---
### Phantom Menace

In this second post of the series of "Transaction Isolation", I would like to discuss about the _committed read_ isolation level and its risks.

```go

...

	ctx1 := context.Background()
	tx1, err := db.BeginTx(ctx1, &sql.TxOptions{Isolation: sql.LevelReadCommitted})
	if err != nil {
		log.Fatalln(err)
	}
	printUsers(tx1)

	go func() {
		ctx2 := context.Background()
		tx2, err := db.BeginTx(ctx2, &sql.TxOptions{Isolation: sql.LevelReadCommitted})
		if err != nil {
			log.Fatalln(err)
			tx2.Rollback()
			return
		}
		_, err = tx2.ExecContext(ctx2, "INSERT INTO users(name) VALUES('Elmo')")
		if err != nil {
			log.Fatalln(err)
			tx1.Rollback()
			return
		}
		tx2.Commit()
	}()

	time.Sleep(1 * time.Second)
	printUsers(tx1)
	tx1.Rollback()
}

func printUsers(tx *sql.Tx) {
	r, err := tx.Query("SELECT name FROM users")
	if err != nil {
		log.Fatalln(err)
		tx.Rollback()
		return
	}
	defer r.Close()
	var name string
	fmt.Print("tx1 sees the following users: ")
	for r.Next() {
		r.Scan(&name)
		fmt.Printf(" %s ", name)
	}
	fmt.Print("\n")
}
```

When running the code, we see that the second transaction, `tx2`, being isolated with a `COMMITED_READ` level, sees the inconsistent values for the users.
In the first `SELECT` query users, it sees `Bert` and `Ernie`, but on the second query, it sees `Bert, Ernie` and `Elmo`.
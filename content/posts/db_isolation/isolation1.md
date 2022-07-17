---
title: "Transactions Isolation #1: Dirty Read"
description: "The first post in a series of posts about database transactions and isolation levels. This one is about read uncommited and dirty reads."
date: 2022-05-10T11:13:37+02:00
draft: false
categories: 
- Development
series:
- 'Transaction Isolation'
tags:
- Database
- Go
- Golang
- Concurrency
---
### Transactions don't socialize

A _Database Transaction_ is an atomic unit of work, sometimes made up of multiple operations.

When a transaction makes multiple changes to the database, either all the changes succeed when the transaction is _committed_, or all the changes are undone when the transaction is _rolled-back_.

_Isolation_ is the 'I' in ACID (a set of properties of a database transactions intend to guarantee data validity)
It ensures that concurrent execution of transactions will obtain the same state that would have been obtained if the transactions were executed sequentially. 

When executing transactions, we can choose differnt levels of isolation, which sets the tradeoff between performance and reliability.

To set an isolation level for a transaction we pass `sql.TxOptions` with the desired isolation level:

```go
...
tx, err := db.BeginTx(context.Background(), &sql.TxOptions{Isolation: sql.LevelReadUncommitted})
...
```


In the following series of blogs I would like to introduce the different isolation level together with their tradeoffs.

Lets use the following `users` table for the next examples:

```sql
CREATE TABLE `users` (
  `id` int NOT NULL AUTO_INCREMENT,
  `name` varchar(30) NOT NULL,
  `balance` int NOT NULL DEFAULT '0',
  PRIMARY KEY (`id`)
)

INSERT INTO users(name, balance) values ("Bert", 100);
INSERT INTO users(name, balance) values ("Ernie", 100);
```

### Reading Gets Dirty
When using the least isolated level - `READ_UNCOMITTED` - one transacation can see another transaction's uncommited changes.
Such phenomena has a name. It is called a _Dirty Read_.

```go
func main() {
	db, err := sql.Open("mysql", "user:pass@tcp(127.0.0.1:3306)/my_db")
	if err != nil {
		log.Fatalln(err)
	}
	defer db.Close()

	ctx1 := context.Background()
	tx1, err := db.BeginTx(ctx1, nil)
	if err != nil {
		log.Fatalln(err)
	}

	_, err = tx1.ExecContext(ctx1, "UPDATE users SET amount = 50 WHERE name = 'Bert'")
	if err != nil {
		log.Fatalln(err)
		tx1.Rollback()
		return
	}

	go func() {
		ctx2 := context.Background()
		tx2, err := db.BeginTx(ctx2, &sql.TxOptions{Isolation: sql.LevelReadUncommitted})
		if err != nil {
			log.Fatalln(err)
			tx2.Rollback()
			return
		}
		r, err := tx2.Query("SELECT amount FROM users WHERE id = 1")
		if err != nil {
			log.Fatalln(err)
			tx2.Rollback()
			return
		}
		defer r.Close()
		var amount int
		for r.Next() {
			r.Scan(&amount)
		}
		fmt.Printf("TX2: Bert has a balance of %d€\n", amount)
	}()

	time.Sleep(1 * time.Second)
	tx1.Rollback()
}

```
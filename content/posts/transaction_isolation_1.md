---
title: "Transactions Isolation #1: Dirty Read"
date: 2022-05-30T11:13:37+02:00
draft: false
---

> "Don't they know we're so afraid? Isolation" _Jonny Cash_

// If someone else will try to update the same data concurrently, then the transaction would behave differently depends on its isolation level.

### Reading Gets Dirty
When using the lowest isolation level - `READ_UNCOMITTED`, a phenomena called _Dirty Read_ can take place. In this phenomena one transacation can see another transaction uncommited changes. Here is a Go example:

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

	_, err = tx1.ExecContext(ctx1, "UPDATE users SET amount = 50 WHERE id = 1")
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
		fmt.Printf("TX2: amount = %d\n", amount)
	}()

	time.Sleep(1 * time.Second)
	tx1.Rollback()
}

```
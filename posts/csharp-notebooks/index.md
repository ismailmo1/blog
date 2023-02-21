---
title: Notebooks as a learning tool for C#
description: Using .NET Interactive and polyglot notebooks in VSCode to experiment with C#
date: 2-11-2023
categories:
  - dotnet
  - csharp
  - learning
  - notebooks
  - vscode
image: polyglot-nb-ext.png
format:
    html:
        toc: true
        toc-location: left
        number-sections: true
---

With .NET Interactive: a .NET based jupyter kernel you can run small snippets of C# (and F#) code in cells and take advantage of a [repl](https://www.digitalocean.com/community/tutorials/what-is-repl) like workflow to explore the language.

# Why notebooks?
My first introduction to coding was in Python via Jupyter Notebooks - this interactive environment with immediate feedback from running each cell was invaluable in my learning, and I still regularly open up an ipython in a terminal when I quickly want to test some behaviour e.g. how do I add things to the start of a list in python again?

Of course doing all your coding in notebooks will leave some pretty big knowledge gaps with things like packaging, project structure, testing and other practices around development (although it can be done in notebooks!), but I still think it's useful to help build understanding around the fundamentals of the language and its ecosystem.

# Getting started

You need to install the [.NET6 SDK](https://dotnet.microsoft.com/download/dotnet/6.0), [VSCode](https://code.visualstudio.com/) and the[Polyglot Notebooks extension](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.dotnet-interactive-vscode).

Then create a `.ipynb` file, pick .NET Interactive as your kernel and start coding:

![](./screen-capture.webm)

You can also read more about how microsoft made this possible in their [devblog](https://devblogs.microsoft.com/dotnet/dotnet-interactive-notebooks-is-now-polyglot-notebooks/).


# C# 101 Challenges
Here's a collection of challenges solved in notebooks. These were taken from the [C# 101 course](https://www.youtube.com/watch?v=qK7tUpaOXi8&list=PLdo4fOcmZ0oVxKLQCHpiUWun7vlJJvUiN&index=11) to build some muscle memory and get some hands on practice with C# syntax

## Combining branches and loops

> See if you can write C# code to find the sum of all integers 1 through 20 that are divisible by 3.

```csharp
int sum = 0;
for (int i=1; i <=20; i++){
    if (i % 3 == 0){
        sum += i;
    }
}
Console.WriteLine(sum);
```

    63


## Working with lists

> Make a list of groceries you need called `groceries`.
> Can you print out "buy \<grocery\>!" for every item?
> What's the 3rd item of the list? Is that the same as `groceries[3]?`
> Make sure to add "toothpaste".
> Remove your least favorite item.
> Now what's third on the list?


```csharp
var groceries = new List<string> {"cheese", "bread", "biscuits", "coffee"};

groceries.Add("toothpaste");
groceries.Remove("biscuits");

foreach (string grocery in groceries){
    Console.WriteLine($"buy {grocery}");
}
```

    buy cheese
    buy bread
    buy coffee
    buy toothpaste

> Make a list of groceries, then sort them! what is the index that 'Carrots' is at?
> Try making a list of numbers! Do they sort in the way you expect?
> Print out the lists that you've made.


```csharp
var groceries = new List<string> {"cheese", "biscuits", "coffee", "carrots"};
int carrotIndex = groceries.IndexOf("carrots");
Console.WriteLine($"carrots at {carrotIndex} before sorting.");

groceries.Sort();
carrotIndex = groceries.IndexOf("carrots");
Console.WriteLine($"carrots at {carrotIndex} after sorting.");
```

    carrots at 3 before sorting.
    carrots at 1 after sorting.

## Fibonacci to 20th number

> Make and print a list that has the first 20 fibonacci numbers.


```csharp
// can use recursion too, but we're practicing lists here
var fibonacciNumbers = new List <int> {1,1};

int idx = 1;
while (fibonacciNumbers.Count <20){
    int currNum = fibonacciNumbers[idx];
    int prevNum = fibonacciNumbers[idx-1];
    int nextNum = currNum + prevNum;
    fibonacciNumbers.Add(nextNum);
    idx ++;
}

foreach(int num in fibonacciNumbers){
    Console.WriteLine(num);
};
```

    1
    1
    2
    3
    5
    8
    13
    21
    34
    55
    89
    144
    233
    377
    610
    987
    1597
    2584
    4181


## Classes


> It has a 10-digit number that uniquely identifies the bank account.
> It has a string that stores the name or names of the owners.
> The balance can be retrieved.
> It accepts deposits.
> It accepts withdrawals.
> The initial balance must be positive.
> Withdrawals cannot result in a negative balance.
> Create a way to list out the list of transactions, including the time and notes.

```csharp
using System.Collections.Generic;

public class Transaction
{
    // Properties
    public decimal Amount { get; }
    public DateTime Date { get; }
    public string Notes
    {
        get;

    }

    // Constructor
    public Transaction(decimal amount, DateTime date, string note)
    {
        this.Amount = amount;
        this.Date = date;
        this.Notes = note;
    }
}

public class BankAccount
{
    public string Number { get; }
    public string Owner { get; set; }
    public decimal Balance
    {
        get
        {
            decimal balance = 0;
            foreach (var item in allTransactions)
            {
                balance += item.Amount;
            }
            return balance;
        }
    }
    private static int accountNumberSeed = 1234567890;
    private List<Transaction> allTransactions = new List<Transaction>();

    public BankAccount(string name, decimal initialBalance)
    {
        this.Owner = name;
        this.Number = accountNumberSeed.ToString();
        accountNumberSeed++;
        MakeDeposit(initialBalance, DateTime.Now, "Initial balance");
    }

    public void MakeDeposit(decimal amount, DateTime date, string note)
    {
        if (amount <= 0)
        {
            throw new ArgumentOutOfRangeException(nameof(amount), "Amount of deposit must be positive");
        }
        var deposit = new Transaction(amount, date, note);
        allTransactions.Add(deposit);
    }

    public void MakeWithdrawal(decimal amount, DateTime date, string note)
    {
        if (amount <= 0)
        {
            throw new ArgumentOutOfRangeException(nameof(amount), "Amount of withdrawal must be positive");
        }
        if (Balance - amount < 0)
        {
            throw new InvalidOperationException("Not sufficient funds for this withdrawal");
        }
        var withdrawal = new Transaction(-amount, date, note);
        allTransactions.Add(withdrawal);
        }
    
    public void ListTransactions()
    {
        //write header
        Console.WriteLine($"Date\t\t\tAmount\t\tNotes");

        foreach (var transaction in allTransactions)
        {
            Console.WriteLine($"{transaction.Date}\t{transaction.Amount}\t\t{transaction.Notes}");
        }
    }
}
```

Thanks to the interactive environment, we can use this class and play about with it:

```csharp
var account = new BankAccount("Ismail", 1000);
Console.WriteLine($"Account {account.Number} was created for {account.Owner} with {account.Balance} dollars");

account.MakeWithdrawal(500, DateTime.Now, "Rent payment"); 
Console.WriteLine(account.Balance);
account.MakeDeposit(100, DateTime.Now, "Friend paid me back");
Console.WriteLine(account.Balance);
```

    Account 1234567890 was created for Ismail with 1000 dollars
    500
    600

```csharp
account.ListTransactions();
```

    Date			Amount		Notes
    22/01/2023 23:36:54	1000		Initial balance
    22/01/2023 23:36:54	-500		Rent payment
    22/01/2023 23:36:54	100		Friend paid me back

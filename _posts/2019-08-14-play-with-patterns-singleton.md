---
layout: playground
title: "Play with Patterns - Singleton"
categories: article
tags: ["kotlin", "design patterns"]
excerpt_separator: <!--more-->
poster: "/assets/patterns/singleton.jpg"
hidden: true
---
I decided to start a series of articles about design patterns. I am not going to describe the patterns in details, because it has been already done. What I am going to do is to allow you to read about the pattern, see how to implemented them and finally, try to do it by yourself right in your browser using Kotlin and see the result.

In this article, I will tell you about the Singleton pattern and then you will try to implement it, so let's start!
<!--more-->

## What is Singleton pattern?
Singleton pattern restricts the instantiation of a class to one object.

## Example
Implementing Singleton pattern in Kotlin is super simple, just create a companion object inside the class and make the constructor private, so no one, except the class itself, could instantiate an instance.
```kotlin
class MyClass private constructor() {
    companion object {
        val instance = MyClas()
    }

    fun foo() { }
}
```

And then use it.
```kotlin
MyClass.instance.foo()
```
## Try it
Let's implement a real-life situation. Imagine that John is an entrepreneur and he needs a license to open a bank account. He goes to the License Department and gets it, then he takes it to the bank to open an account. The bank checks the license by asking the License Department and if everything is fine, opens an account.

In this situation, the License Department is a Singleton, because if Jong took the license from the License Department A and the bank checkes John's license in the License Department B. The license can not be verified as it has been issued by a different organization. As a result, the bank does not open an account.

Here is an implementation of the process described above but it does not work correctly. You need to fix it by using the Singleton pattern where it is required.

```kotlin-playgound
import java.util.*

data class Entrepreneur(val name: String)

data class License(val id: String, val owner: Entrepreneur)

data class BankAccount(val id: String, val owner: Entrepreneur)

class LicenseDepartment {
    private val depSignature = "#${UUID.randomUUID().toString().substring(0, 5)}#"

    fun issueLicense(person: Entrepreneur): License =
        License(depSignature + UUID.randomUUID().toString(), person)

    fun checkLicense(person: Entrepreneur, licence: License): Boolean =
        licence.owner == person && licence.id.contains(depSignature)
}

class Bank {
    private val licenceDepartment: LicenseDepartment = LicenseDepartment()

    fun openAccount(owner: Entrepreneur, license: License): BankAccount {
        if (licenceDepartment.checkLicense(owner, license)) {
            return BankAccount(UUID.randomUUID().toString(), owner)
        }
        throw IllegalStateException("The license did not pass the verification")
    }
}

fun main(args : Array<String>) {
    val licenseDepartment = LicenseDepartment()
    val bank = Bank()
    val john = Entrepreneur("John")
    val license = licenseDepartment.issueLicense(john)
    val account = bank.openAccount(john, license)

    println(account)
}
```
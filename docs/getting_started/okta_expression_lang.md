---
layout: docs_page
title: Okta Expression Language
---

* Will be replaced with the ToC
{:toc .list-unstyled .toc}

## Overview

> Okta EL is currently only supported on the Profile Mappings Page and in the SAML App Wizard.

This document details the features and syntax of Okta's Expression Language.  Expressions allow you to reference, transform and combine attributes before you store them on a user profile or before passing them to an application for authentication or provisioning.  For example, you might use a custom expression to create a username by stripping @company.com from an email address.  Or you might combine `firstName` and `lastName` attributes into a single `displayName` attribute.

This document will be updated over time as new capabilities are added to the language.  Okta's expression language is based on [SpEL](http://docs.spring.io/spring/docs/3.0.x/reference/expressions.html) and uses a subset of functionalities offered by SpEL.

## Referencing User Attributes
When you create an Okta expression, you can reference any attribute that lives on an Okta user profile or App user profile.

###Okta user profile
Every user has an Okta user profile.  The Okta user profile is the central source of truth for a user's core attributes.  To reference an Okta user profile attribute, just reference `user` and specify the attribute variable name.


Syntax  | Definitions | Examples
-------- | ---------- | ------------
`user.$attribute` | `user` reference to the Okta user<br>`$attribute` the attribute variable name | user.firstName<br>user.lastName<br>user.username<br>user.email

###Application user profile
In addtion to a Okta user profile, all users have separate Application user profiles for each of their applications.  The Application user profiles are used to store application specific information about a user like their application username or their user role.  To reference an App user profile attribute, just specify the application variable and the attribute variable in that application's App user profile. In specifying the application you can either name the specific application you're referencing or use an implicit reference to an in-context application.

Syntax  | Definitions | Examples
-------- | ---------- | ------------
`$appuser.$attribute` | `$appUser` explicit reference to specific app<br>`$attribute` the attribute variable name | zendesk.firstName<br>active_directory.managerUpn<br>google_apps.email
`appuser.$attribute` | `appUser` implicit reference to in-context app<br>`$attribute` the attribute variable name | appUser.firstName

> With Universal Directory, there are about 30 attributes in the base Okta profile and you can add any number of custom attributes.  All App user profiles have a username attribute and possibly others depending on the application.   To find a full list of Okta user and App user attributes and their variable names, go to People > Profile Editor.  If you're not yet using Universal Directory, contact your Support or Professional Services team.

## Referencing Application and Organization Properties
In addition to referencing user attributes, you can also reference Application properties and properties of your Organzation.  To reference a particular attribute, just specify the appropriate binding and the attribute variable name.  Here are some examples:

###Application properties

Syntax | Definitions | Examples
------ | ---------- | ------------
`$app.$attribute` | `$app` explicit reference to specific app instance<br>`$attribute` the attribute variable name | google_apps_app.domain<br>zendesk_app.companySubDomain
`app.$attribute` | `app` implicit reference to in-context app instance<br>`$attribute` the attribute variable name | app.domain<br>app.companySubDomain

###Organization properties

Syntax  | Definitions | Examples
-------- | ---------- | ------------
`org.$attribute` | `org` reference to Okta org<br>`$attribute` the attribute variable name | org.domain


> For a full list of Application and Org attributes, contact your Support or Professional Services team.


## Functions

Okta offers a variety of functions to manipulate attributes or properties to generate a desired output.  Functions can be combined and nested inside a single expression.

### String Functions

Function  | Example | Input | Output
-------- | --------- | -------| --------
`toUppercase(string)` | `toUppercase(source.firstName)` | Alexander | ALEXANDER
`toLowercase(string)` | `toLowercase(source.firstName)` | AlexANDER | alexander
`substringBefore(string, string)` | `substringBefore(user.email, '@')` | alex@okta.com | alex
`substringAfter(string, string)` | `substringAfter(user.email, '@')` | alex@okta.com | @okta.com
`substring(string, int, int)` | `substring(source.firstName, 1, 4)` | Alexander | lex

### Array Functions

Function  | Return Type | Example | Output
-------- | ---------| --------- | --------
`Arrays.add(array, value)` | Array | Arrays.add({10, 20, 30}, 40) | {10, 20, 30, 40}
`Arrays.remove(array, value)` | Array | Arrays.remove({10, 20, 30}, 20) | {10, 30}
`Arrays.clear(array)` | Array | Arrays.clear({10, 20, 30}) | { }
`Arrays.flatten(list of values)` | Array | Arrays.flatten(10, {20, 30}, 40) | {10, 20, 30, 40}
`Arrays.contains(array, value)` | Boolean | Arrays.contains({10, 20, 30}, 10) | true
 |  | Arrays.contains({10, 20, 30}, 50) | false
`Arrays.size(array)` | Integer | Arrays.size({10, 20, 30}) | 3
 |  | Arrays.size(NULL) | 0
`Arrays.isEmpty(array)` | Boolean | Arrays.isEmpty({10, 20}) | false
 |  | Arrays.isEmpty(NULL) | true


### Conversion Functions

Function  | Return Type | Example | Input | Output
-------- | ---------| --------- | -------| --------
`Convert.toInt(string)` | Integer | `Convert.toInt(val)` | String val = '1234' | 1234
`Convert.toNum(string)` | Double | `Convert.toNum(val)` | String val = '3.141' | 3.141

### Directory and Workday Functions

Function  | Description
-------- | ---------
`hasDirectoryUser()` | Checks whether the user has an Active Directory assignment and returns a boolean
`hasWorkdayUser()` | Checks whether the user has a Workday assignment and returns a boolean
`findDirectoryUser()` | Finds the Active Directory App user object and returns that object, or null if the user has more than one or no Active Directory assignments
`findWorkdayUser()` | Finds the Workday App user object and returns that object, or null if the user has more than one or no Active Directory assignments

The functions above are often used in tandem to check whether a user has an AD or Workday assignment, and if so return an AD or Workday attribute.  See the 'Popular Expressions' table below for some examples.

## Constants and Operators

Common Actions  | Example
----------------| --------
Refer to a `String` constant | 'Hello world' 
Refer to a `Integer` constant | 1234
Refer to a `Number` constant | 3.141
Refer to a `Boolean` constant | true
Concatenate two strings | `user.firstName`+`user.lastName`
Concatenate two strings with space | `user.firstName`+" "+`user.lastName`
Ternary operator example:<br>If group code is 123, assign value of Sales, else assign Other | user.groupCode==123?'Sales':'Other'

## Popular Expressions

Sample user data:

* Firstname = Winston
* Lastname = Churchill
* Email = winston.churchill@gmail.com
* Login = winston.churchill@gmail.com

Value to Obtain | Expression  | Example Output | Explanation
---------- | ---- | ----- | ---------------
Firstname | `user.firstName` | Winston | Obtain the value of users's firstname attribute.
Firstname + Lastname | `user.firstName + user.lastName` | WinstonChurchill | Obtain Firstname and Lastname values and append each together.
Firstname + Lastname with Separator | `user.firstName + "." + user.lastName` | Winston.Churchill | Obtain Firstname value, append a "." character. Obtain and append the Lastname value
First Initial + Lastname | `substring(user.firstName, 0, 1) + user.lastName` | WChurchill | Obtain Firstname value. From result, retrieve characters greater than position 0 thru position 1, including position 1. Obtain and append the Lastname value
First Initial + Lastname with Limit | `substring(user.firstName, 0, 1) + substring(user.lastName, 0, 6)` | WChurch | Obtain Firstname value. From result, retrieve 1 character starting at the beginning of the string. Obtain Lastname value. From result, retrieve characters greater than position 0 thru position 6, including position 6.
Lower Case First Initial + Lower Case Lastname with Separator | `toLowerCase(substring( user.firstName, 0, 1)) + "." + toLowerCase(user.lastName)` | w.churchhill | Obtain Firstname value. From result, retrieve characters greater than position 0 thru position 1, including position 1. Convert result to lowercase. Append a "." character. Obtain the Lastname value. Convert to lowercase and append.
Email Domain + Email Prefix with Separator | `toUpperCase(substringBefore( substringAfter(user.email, "@"), ".")) + "\" + substringBefore( user.email, "@")` | GMAIL\winston.churchill | Obtain Email value. From result, parse everything after the "@ character". From result, parse everything before the "." character. Convert to uppercase. Append a backslash "\" character. Obtain the email value again. From result, parse for everything before the "@" character.
Email Domain + Lowercase First Initial and Lastname with Separator | `toUpperCase(substringBefore( substringAfter(user.email, "@"), ".")) + "\" + toLowerCase(substring( user.firstName, 0, 1)) + toLowerCase(user.lastName)` | GMAIL\wchurchill | Obtain Email value. From result, parse everything after the "@ character". From result, parse everything before the "." character. Convert to uppercase. Append a backslash "\" character. Obtain the Firstname value. From result, retrieve characters greater than position 0 thru position 1, including position 1. Convert it to lowercase. Obtain the Lastname value and convert it to lowercase.
Static Domain + Email Prefix with Separator | `"XDOMAIN\" + toLowerCase(substring( user.firstName, 0, 1)) + toLowerCase(user.lastName)` | XDOMAIN\wchurchill | Add "XDOMAIN" string. Append a backslash "\" character. Obtain the Firstname value. From result, retrieve characters greater than position 0 thru position 1, including position 1. Convert it to lowercase. Obtain the Lastname value. Convert it to lowercase.
Workday ID | `hasWorkdayUser()?findWorkdayUser().employeeID:null` | 123456 | Check if user has a Workday assignment, and if so, return their Workday employeeID.
Active Directory UPN | `hasDirectoryUser()?findDirectoryUser().managerUPN:null` | bob@okta.com | Check if user has an Active Directory assignment, and if so, return their Active Directory manager UPN.

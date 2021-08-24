## Table of Contents

- [Introduction](#introduction)
- [Terminology](#terminology)
  - [Program](#program)
  - [Plan](#plan)
  - [Requirement](#requirement)
- [How it Works](#how-it-works)
- [The Context Free Language](#the-context-free-language)
  - [Layer 1: Course Set Definition](#layer-1-course-set-definition)
    - [Ranged Course Set (or simply Course Set)](#ranged-course-set-or-simply-course-set-)
    - [Fixed Course Set](#fixed-course-set)
  - [Layer 2: Functions](#layer-2-functions)
    - [`grade`](#-grade-)
    - [`hasTaken`](#-hastaken-)
    - [`gradeAny`](#-gradeany-)
    - [`hasTakenAny`](#-hastakenany-)
    - [`avg`](#-avg-)
    - [`gpa`](#-gpa-)
    - [`hasTakenCrd`](#-hastakencrd-)
    - [`manualCheck`](#-manualcheck-)
  - [Layer 3: Conjunction](#layer-3-conjunction)
    - [Logical Operators](#logical-operators)
    - [Order of Precedence](#order-of-precedence)
- [The Evaluation Tool in Details](#the-evaluation-tool-in-details)
  - [Form's Input Fields](#form-s-input-fields)
  - [Modes of Evaluation](#modes-of-evaluation)
    - [1. Separate Plans](#1-separate-plans)
    - [2. Merged Requirements](#2-merged-requirements)
  - [The Requirements Set](#the-requirements-set)
  - [Evaluation Results (Unsaved Applications)](#evaluation-results-unsaved-applications-)
  - [List of Saved Applications](#list-of-saved-applications)
  - [Application Details](#application-details)
    - [1. Evaluation Details](#1-evaluation-details)
    - [2. Application Info](#2-application-info)
    - [3. Evaluation Result & Final Decision](#3-evaluation-result---final-decision)
    - [4. Display Controllers](#4-display-controllers)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

---

## Introduction

Advizer is a web-app designed and implemented for the Faculty of Arts and Sciences at the American University of Beirut (AUB). It is a tool that evaluates AUB student transcripts against any set of requirements provided by the user. Although it was implemented in coordination with the Students Services Office at FAS, its applicability spans across all AUB faculties and departments. The typical users are internal tranfer committees and academic advisers.

Advizer is designed to be independent from AUB's internal system. It does not require access to AUB records. All the input necessary for any evaluation is provided directly by the user.

## Terminology

The following terms are used across the application:

#### Program

A `Program` represents a major at AUB. Examples include _Computer Science_, _Physics_, _Biology_, etc...

#### Plan

A `Plan` represents a grouping for a set of requirements. Each `Program` contains a set of plans.

For example, within the _Computer Science_ program, we can have _Graduation Requirements (2019-2020)_, _Transfer Requirement (2019-2020)_, _Graduation Requirements (2020-2021)_, _Transfer Requirement (2020-2021)_, etc...

#### Requirement

A `Requirement` is a single rule that is taken from the AUB catalog and rewritten in Advizer terms. Each `Requirement` has three fields:

1. `title` — a meaningful title provided by the user (required)
2. `description` — usually being the catalog's wording copied as-is (optional)
3. `rule` — the equivalent form of the catalog's wording written in Advizer's context-free language (required)

Requirements are at the heart of the application. A `Requirement` is valid if, and only if, it has a `title` and a `rule` AND the `rule` is written in valid syntax.

## How it Works

As mentioned earlier, the `rule` within a `Requirement` must be written in valid Advizer syntax.

To achieve its goal of independence from the AUB system, Advizer has its own custom context-free language that is designed to transform any university requirement into syntax that can be interpreted into evaluation criteria for any given transcript.

For the same purpose also, transcripts are not acquired from AUB through any API. They are expected to be provided by the user. Users are expected to either download the unofficial transcripts as _.html_ files to be uploaded to the tool, or copy & paste the page source of the unofficial transcripts.

With these notes in mind, this is how Advizer works:

1. The user creates a set of _programs_
2. Within each `Program`, the user creates a set of _plans_
3. Within each `Plan`, the user creates a set of _rquirements_ corresponding to that `Plan`
4. On the _Evaluation Tool_ page, the user provides at least one transcript, and at least one `Plan` and runs the evaluation
5. Evaluation results are considered unsaved _applications_ with an `Application` representing the evaluation of one transcript against one set of _requirements_
6. Users are free to select and save as many _applications_ to the database or simply view and discard them

---

## The Context Free Language

The language implemented for Advizer allows the user to represent requirements as functions with parameters.

The specific syntax rules ensures that requirements can be processed for transcript evaluation. These rules, referred to as the grammar, decide whether the syntax used is legal or not. Users cannot save a requirement with invalid syntax.

All requirements are represented as Boolean expressions that evaluate to `true` of `false`. Each expression includes a function. Most functions require a course-set to be its parameter.

With these notes in mind, the grammar can roughly be represented as three layers. Following are these layers starting from the lowest level:

### Layer 1: Course Set Definition

As mentioned earlier, most functions require a cours-set to be passed as a parameter. There are two types of course sets:

> Before discussing course sets, note that a _course_ is defined as: `subject code` where the `subject` can either be an AUB course subject like `CMPS, ENGL, ARAB, etc...` or the keyword `allsub` which stands for _"all subjects"_.

#### Ranged Course Set (or simply Course Set)

Provides a range that can be resolved to any number of courses. This range can either be:

- A single course
  - ex: `cmps 200`
- A list of courses separated by commas
  - ex: `cmps 200, cmps 212, nfsc 221`
- A range of course codes belonging to the same subject
  - ex: `cmps 200-212` (Any cmps course with the code between `200` and `212` inclusive)
- A range in the format `subject op code`
  - `op` is any of the following operators: `>`, `>=`, `<` , `<=`, `==`, `!=`
  - ex: `cmps >= 230`
- Any combination of the preceding formats separated by commas
  - ex: `cmps 200, cmps 210-212, cmps >= 230`

#### Fixed Course Set

This is a subset of the [_Ranged Course Set_](#ranged-course-set). A fixed course set is simply one or more courses separated by commas.

---

### Layer 2: Functions

Within Advizer's language, a requirement is represented as a function. Following are the supported functions:

#### `grade`

- Checks if the grade of _each_ course within a provided course-set meets a given range
- Written in the form: `grade[fcs] op LG` such that:
  - `fcs` stands for [_fixed course-set_](#fixed-course-set)
  - `op` is one of the following operators: `>`, `>=`, `==`
  - `LG` is a letter grade ranging from `D-` to `A+` (case insensitive)
- Example: `grade[cmps 200, cmps 212] >= B` is equivalent to: "the grade of each of _cmps 200_ and _cmps 212_ is greater than or equal to _B_"

> Note that the grade _P_ is not a technically counted as a grade. When compared, it can be considered both greater than and equal to any other grade.
> Thus, if the transcript lists the grade of CMPS 200 as "P" and the rule is `grade[cmps 200] >= B` the expression will evaluate to `true`.

#### `hasTaken`

- Checks if _each_ of the courses within a provided course-set has been completed
- Written in the form: `hasTaken[fcs]` such that:
  - `fcs` stands for [_fixed course-set_](#fixed-course-set)
- Example: `hasTaken[cmps 200, cmps 212]`

> Note that `hasTaken[fcs]` is equivalent to writing `grade[fcs] >= D`

#### `gradeAny`

- Checks if the grade of `x` number of courses within a provided course-set meets a given range
- Written in the form: `gradeAny[x][cs] op LG` such that:
  - `x` is any positive integer
  - `cs` is a course-set that represents a range of courses (more on its syntax later)
  - `op` is one of the following operators: `>`, `>=`, `==`
  - `LG` is a letter grade ranging from `D-` to `A+` (case insensitive)
- Example: `gradeAny[2][cmps] >= B` is equivalent to: "the grade of at least _2_ courses of the subject _CMPS_ is greater than or equal to _B_"

#### `hasTakenAny`

- Checks if a given number of courses within a provided course-set have been completed
- Written in the form: `hasTaken[x][cs]` such that:
  - `cs` stands for [_course set_](#ranged-course-set-or-simply-course-set)
- Example: `hasTakenAny[2][cmps >= 230]`

> Note that `hasTakenAny[x][cs]` is equivalent to writing `gradeAny[x][cs] >= D`

#### `avg`

- Checks if the average of a provided course-set meets a given range
- If the no courses on the transcript are within the provided course-set, the evaluation resolves to `true` - The semantics of this function can be written as "if these courses are taken, there average should be..."
- Courses with a _P/NP_ grade or with a repeat status _A_ are **not** included in the average computation
- Written in the form: `avg[cs] op x` such that:
  - `cs` is a course-set that represents a range of courses (more on its syntax later)
  - `op` is one of the following operators: `>`, `>=`, `==`
  - `x` is a positive number in the range [0.00-4.00] or [40.00-100.00] (decimals are optional)

#### `gpa`

- Checks if the GPA on a transcript meets a given range
- Written in the form: `gpa op x` such that:
  - `op` is one of the following operators: `>`, `>=`, `==`
  - `x` is a positive number in the range [0.00-4.00] or [40.00-100.00] (decimals are optional)

#### `hasTakenCrd`

- Checks if the student has completed a given number of credits
- Written in the form: `hasTakenCrd[x]` such that:
  - `x` is any positive integer

#### `manualCheck`

- Used when the catalog requirement requires human interaction or when the current syntax is not capable of representing the requirement
- Written in the form: `manualCheck[comment]` such that:
  - A `comment` **must** be surrounded by either double quotes `"` or single quotes `'`
- The output of this expression is always `null` (neither `true` nor `false`)

---

### Layer 3: Conjunction

#### Logical Operators

The language supports the three fundamental Boolean operators: `and`, `or`, & `not`.

Any number of expressions written with the functions listed above can be combined into a single expression using `or` & `and`; or it can be negated using `not`.

#### Order of Precedence

The order of precedence of the Boolean operators is as follows: `not` > `and` > `or`.

In other words if we have the scenario `exp1 or exp2 and exp3`, then `exp2 and exp3` will be evaluated first.

The language supports parenthesis and they override the order of precendence. For example, if we have the scenario `(exp1 or exp2) and exp3`, then `exp1 or exp2` will be evaluated first.

## The Evaluation Tool in Details

### Form's Input Fields

The _Evaluation Tool_ page has one main form with three input fields:

1. The _plans_ to be used in the evaluation
2. The transcripts to be evaluated (.html file or direct input of html code)
3. The list of acceptable GE Courses for the current evaluation (.csv files only)

All three input fields can take multiple values (multiple plans, multiple transcripts, and multiple GE courses files) at once.

### Modes of Evaluation

There are two modes of evaluation:

#### 1. Separate Plans

In this mode, _each_ of the selected plan is used separately to evaluate _each_ of the provided transcripts.

When this mode of evaluation is chosen, the total number of _applications_ created is the product of the number of transcripts and the number of _plans_ selected.

#### 2. Merged Requirements

In this mode, _all_ the selected plans have their requirements merged into a single list, and that list of requirements is used to evaluate _each_ of the provided transcripts.

When this mode of evaluation is chosen, the total number of applications created is the same as the number of transcripts provided; regardless of how many plans are merged together.

### The Requirements Set

Upon selecting evaluation plans, the 'Requirements Set Table' will be filled with a list of requirements that are either grouped or presented as a single list depending on the evaluation mode.

Regardless of the evaluation mode, the user is able to select/deselect any number of the requirements in the list. Only the selected requirements are used in the evaluation.

The button to toggle between evaluation modes is found at the top right of the Requirements Set Table.

---

### Evaluation Results (Unsaved Applications)

The evaluation of a transcript against a set of requirements is referred to as an application. Applications are not automatically saved upon evaluation. Users can select any number of the resulting applications to be stored in their account.

---

### List of Saved Applications

Upon saving an evaluation result, the following process occurs in order:

- An `Application ID` is computed using information about the applicant, the selected program, the selected plan, the selected requirements.
- If the resulting ID is found in the database, then the found record is updated with the new evaluation results and the `final decision` field is reset to `pending`
- Otherwise, a new application is created with the corresponding data and the `final decision` field set to `pending`
- Once an application is created, it is encrypted using a 256-bit key that is uniquely generated for each user and never shared with the server, and then saved encrypted in the database.

> Note here that applications are _only_ accessible to their originator (even the system's admin cannot decrypt these records as they are treated as private information since they contain student names, desired major, and course grades).

> Because the encryption/decryption key is only accessible to the user, resetting a forgotten password will lead to deleting the stored applications. This is by design to protect private information. The programs and plans will not be affected.

---

### Application Details

Each application, saved or unsaved, is viewable in a popup window. If the application is already saved, the top-right button allows viewing it in its own window. Otherwise, the top-right button is used to save the application.

There are **_four_** main sections in this page:

#### 1. Evaluation Details

This section has to parts: the summary cards, and the detailed evaluation tree.

The evaluation shows the selected requirements with an icon representing one of the three states of requirements' evaluation:

- Satisfied ✅
- Unsatisfied ❌
- Undecided ❓

Requirements that are formulated as a conjunction of multiple functions, are expandable to show the evaluation result of each underlying component to accurately convey the root cause behind the final verdict.

#### 2. Application Info

This section contains details about the transcript and the plan. The student's name and ID are extracted automatically from the HTML transcript uploaded.

Details about the evaluation mode and the chosen plan are viewable below the applicant's details. Evaluation mode will either be `Merged Requirements` or `Separate Plans`.

The title of the selected plan will always be `N/A` if the evaluation mode is `Merged Requirements`.

If the evaluation mode is `Separate Plans` there are two cases for the plan's title displayed:

- If one or more of the plan's requirements are _deselected_, then the plan's title is appended with the suffix: "(customized)"
- If all the plan's requirements were selected, the plan's title is displayed as-is

#### 3. Evaluation Result & Final Decision

_Evaluation result_ is the computed result by the evaluation tool. It has three possible values: `incomplete`, `complete`, and `undecided`. The value cannot be changed manually by the user.

_Final decision_ \*\*\*\*is the ultimate decision, made by the user, regarding the given application. It was established to override evaluation results in exceptional cases, and to give a final verdict when the evaluation result is `undecided`.

#### 4. Display Controllers

There are **_four_** settings to control how evaluation results are displayed. All settings are **_on_** by default.

1. **Collapsed view:**

   When **_off_**, expands the entire evaluation tree

   When **_on_**, restores the evaluation tree to its previous state of expanded & collapsed items

2. **Fewer ticks:**

   When **_off_**, displays the icon of the evaluation result for every line.

   When **_on_**, displays icons only at the highest and lowest levels of the evaluation tree only

   > Sometimes showing all icons when several requirements are expanded leads to a confusing view

3. **Minimal details:**

   When **_off_**, shows comments for every single evaluation result.

   When **_on_**, comments for unsatisfied requirements only are shown.

4. **Display titles:**

   When **_off_**, shows the requirement's rule at the root level as well as the sublevels (if any)

   When **_on_**, shows the requirement's title at the root level, and rules at the sublevels (if any)

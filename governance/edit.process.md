# OWIN Governance - Editing Process

**Version**
draft-editing-process-01

**Authors**

| Author Name | Contact Info |
|-------------|--------------|
| OWIN Working Group | [OWIN Working Group on GitHub](https://github.com/owin/owin) |
| Eddie Garmon | eddie_garmon@hotmail.com |

**Copyright Notice**
Copyright (c) OWIN Working Group, and all listed authors (2014).

**Last updated**
17 October 2014  

## Abstract

This document defines the editing and versioning process for the set of
specification documents that define OWIN and its extensions.

## Status

This is a working draft.

This document is a submission to the OWIN Working Group and is valid for a
period of six months and may be updated, replaced, or obsoleted by other
documents at any time. It is inappropriate to use working drafts as reference
material or to cite them other than as "work in progress"

This working draft will expire on 17 April 2015.

## Contents

1. [Introduction](#intro)
1.1. [Requirements](#requirements)
1.2. [Terminology](#terminology)
1. [Draft Naming Rules](#naming)
1. [Markdown Format Rules](#format)
1. [Submission and Revisions](#submission)
1. [Versioning and Publication](#versioning)
1. [Full Copyright Statement](#copyright)

<a name="intro"></a>
## 1. Introduction

This document defines the working process for editing any document within the
OWIN specification set. This includes the appropriate draft naming policy, git
branching policy, Markdown document formatting rules, and the revision and
approval process. This process is to make it clear what work is currently
underway, and to provide an easy and consistent manner to address works in
progress for discussion and voting.

<a name="requirements"></a>
### 1.1 Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

<a name="terminology"></a>
### 1.2 Terminology

This specification uses a number of terms to refer to the roles played by
participants in the editing process.

*OWIN Spec* - The set of all documents that define and describe the core
  functionality, all extensions, and all addendum.

*author* - A member of the working group that creates, amends, or edits one or
  more documents in the OWIN Spec.

*draft* - A revision of one or more documents that are to be put forth to the
  working group for review, comment, edit, and vote.

*publication* - A draft that is accepted by the OWIN Working Group.

<a name="naming"></a>
## 2. Draft Naming Rules

All drafts SHOULD exist for one specific purpose, and multiple purposes SHOULD NOT
be incorporated into the same draft. It is OK and allowed to edit multiple 
documents at the same time for the same purpose, i.e. to comprehensively define
a change that affects multiple documents in the same working set.

Drafts should be named according to the following convention:
```
draft-{purpose}-{revision}
```
Where 'draft-' is a standard prefix. {purpose} should be replaced with a short
descriptive purpose of the draft in a few words. {revision} should be replaced
with the current revision in 2 digit format, i.e. 01, 07, or 12. The revision
MUST always start at 01, and MUST be incremented by one only after a draft is
put forward to a vote. [See submission and revisions](#submission).

Good examples would be:
- draft-core-middleware-01
- draft-core-middleware-23
- draft-extension-websockets-04

Bad examples would be:
- OWIN-1.1
- master

All drafts MUST be contained in their own git branch and each branch SHOULD be
named the same as the draft name.

<a name="format"></a>
## 3. Markdown Format Rules

All documents put forth to the working group MUST be in Markdown format with the
follow stylistic rules:
- GitHub flavored markdown is acceptable
- Content should be within the ASCII codepage
- Lines should have hard breaks at no more than 85 characters
- Lines should not have trailing white space
- All documents should have the following items in the header
	- Version
	- Authors
	- Copyright Notice
	- Date of last update
- All documents should have a full copyright statement at the end

<a name="submission"></a>
## 4. Submission and Review

After a draft has been created and edited by one or more authors, they submit a
pull request to the OWIN Working Group. This request will be pulled into a draft
branch for the community as a whole to review, comment upon, and make additional
changes via pull request.

At least one week (seven days) after the Working Group has committed the pull
request and one week (seven days) after the date of the last edit, a vote MAY be
requested on the draft. If the vote passes, the draft will have an appropriate
[semantic version](http://semver.org/) assigned to it, and it will be merged
into master and published.

If a vote does not pass, or if a critical issue is raised during the voting
process, the draft revision MUST be incremented and have the issue(s) addressed
before being brought to vote again. This may happen as many times as necessary.

All draft branches that are not accepted MUST remain in the repository.

All draft branches that are accepted will be merged into master and all 
associated revision branches will be removed.

<a name="versioning"></a>
## 5. Versioning and Publication

The appropriate semantic version will be determined after the draft is complete.
Authors should not worry about the next version number to be assigned, but
instead should focus on getting the specification correct, even if it introduces
breaking changes from the last published version.

After a vote of acceptance by the working group the following edits MUST take
place on the draft branch before merging:
- The correct version is assigned
- The status is updated to reflect the date of adoption
- The document authors should be verified and appended accordingly
- The names of the members of the working group that participated in the vote
   should be documented (but not their vote)

After these bookkeeping edits are made, that branch will be merged into master.

All documents on the master branch will be considered published.

<a name="copyright"></a>
## 6. Full Copyright Statement

Copyright (C) OWIN Working Group and all listed authors.  All Rights Reserved.

This document and translations of it may be copied and furnished to
others, and derivative works that comment on or otherwise explain it
or assist in its implementation may be prepared, copied, published
and distributed, in whole or in part, without restriction of any
kind, provided that the above copyright notice and this paragraph are
included on all such copies and derivative works.  However, this
document itself may not be modified in any way, such as by removing
the copyright notice or references to the OWIN Working Group or other
authors, except as required to translate it into languages other than
English.

The limited permissions granted above are perpetual and will not be
revoked by the OWIN Working Group and authors or its successors or assigns.

This document and the information contained herein is provided on an
"AS IS" basis and THE OWIN WORKING GROUP AND ALL AUTHORS DISCLAIMS ALL
WARRANTIES, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO ANY WARRANTY THAT
THE USE OF THE INFORMATION HEREIN WILL NOT INFRINGE ANY RIGHTS OR ANY IMPLIED
WARRANTIES OF MERCHANTABILITY OR FITNESS FOR A PARTICULAR PURPOSE.


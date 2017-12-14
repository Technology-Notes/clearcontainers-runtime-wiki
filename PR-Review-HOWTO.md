* [High level considerations](#high-level-considerations)
    * [Security](#security)
    * [Reliability](#reliability)
    * [Performance](#performance)
    * [Maintenance](#maintenance)
    * [Usability](#usability)
* [Specifics to consider](#specifics-to-consider)
    * [Basics](#basics)
    * [Layout and formatting](#layout-and-formatting)
    * [Design](#design)
    * [Code comments](#code-comments)
    * [Documentation](#documentation)
    * [Logging and Debugging](#logging-and-debugging)
    * [Testing](#testing)
    * [Environments](#environments)
    * [Upgrades](#upgrades)
* [Mandatory Checks](#mandatory-checks)
---                                       

This document attempts to capture some of the points to consider when reviewing a PR to ensure code quality is kept as high as possible.

## High level considerations

### Security

- Could the change introduce a potential issue?
  - What additional checks would resolve it?

### Reliability

- What happens on failure? (Consider **every** possible failure)

### Performance

- Will this change negatively impact performance?
- Can any calls block? If so, there should be a log call before the blocking call explaining what is about to be done.

### Maintenance

- How easy would this code be to maintain
  - By an experienced developer?
  - By a developer with none / almost no experience of the codebase?

### Usability

- Does the change improve or worsen usabililty?
- Could it be made any better?

## Specifics to consider

### Basics

- Consider all inputs and outputs.
- What resources are being used? (memory, disk, *etc*) Are they released?
- Are all return values and arguments checked?
- Are there any magic numbers?

### Layout and formatting

- Is formatting and naming consistent?
- Are there any spelling mistakes ("typos")?
- Do all new files contain the required licence header?

### Design

- Could the code be simplified / refactored?
  - Is the code "too clever"? Overly clever code tends to be fragile: reject it unless there is a **very** good documented reason!
- Is there a better solution to the problem?
- Is there any duplication?
- What assumptions does the code make? Are they valid? Are they documented?
- Does the code make sense?
  - Could someone with minimal exposure to the codebase guess at what it is doing? (hint: they should be able to).

### Code comments

- Is the code sufficiently well commented?
- Are all functions and function parameters documented (or for `golang` programs, is they purpose obvious?)
- Does this change require an update to the `README`(s) / architecture docs?
- Are any lines commented out? If so, they should be **removed**!
- Are there any `FIXME`/`TODO`'s? If so, they should be replaced by a full URL to an issue number tracking the work that needs to be done.

### Documentation

- Does the document update need to be applied to any **other** documents?
  For example, when a PR updates an installation guide, those same changes may need to be applied to one or more of the other installation guides!

- Does the PR make any of the following changes? If so, it may well need a documentation change:
  - new programs / scripts added or removed.
  - new program / script options added / removed.
  - new config options added or removed.
  - new features added / removed.
  - existing bug(s) fixed.

### Logging and Debugging

- Could this code be debugged from a logfile if it failed?
- Are error messages and log calls useful and comprehensive? Ensure sensitive information is **not** displayed though.

### Testing

- Has the code been written to be easy to test?
- Are there new tests? If not, why?

### Environments

- Will the code work on all distributions?
- What environments will this code run in?
- Which user will the code run as?

### Upgrades

- If the code logs state, how will upgrades be handled?

## Mandatory Checks

- Provide constructive comments on the code.
  - If you like some aspect particularly, say so! :smile:
- Ensure all automated checks pass.
- Ensure the PR contains atleast one `Fixes #XXX` commit message referencing an issue that this PR fixes.
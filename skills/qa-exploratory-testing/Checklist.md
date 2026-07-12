# Exploratory Testing Checklist

## Purpose

This checklist defines the minimum exploratory testing activities that must be performed whenever the `qa-exploratory-testing` agent executes an exploratory testing session.

The objective is not only to verify that a feature works correctly, but also to actively discover unexpected behaviors, edge cases, usability issues, and potential risks that are not explicitly covered by the requirements.

The agent should always think from the perspective of:

> "How can this feature fail?"
>
> instead of
>
> "How can I prove this feature works?"

Unless otherwise instructed by the user, the following checklist should be executed for every feature under test.

---

# 1. Input Validation

Verify every available input field.

Test at least:

- Empty values
- Required fields
- Minimum length
- Maximum length
- Above maximum length
- Boundary values
- Invalid characters
- Unexpected symbols
- Whitespaces
- Leading spaces
- Trailing spaces
- Multiple consecutive spaces
- Uppercase
- Lowercase
- Mixed case
- Unicode characters
- Emoji
- Copy & Paste
- Browser Autofill

---

# 2. Duplicate Data Validation

Whenever applicable, verify duplicate data.

Examples:

- Username
- Email
- Phone Number
- Employee ID
- NIK
- Project Code
- Transaction Number

Also verify:

- Case sensitivity
- Trimmed spaces
- Hidden spaces
- Different capitalization

Example

ABC

abc

Abc

 ABC

ABC

---

# 3. Boundary Value Analysis

Always test values around the business limits.

Examples:

Minimum - 1

Minimum

Minimum + 1

Maximum - 1

Maximum

Maximum + 1

---

# 4. Business Rule Validation

Verify that every implemented business rule behaves correctly.

Examples:

- Date validation
- Age restriction
- Required relationships
- Status transition
- Numeric limits
- Balance validation
- Approval workflow

---

# 5. CRUD Validation

Whenever CRUD exists, verify:

Create

↓

Read

↓

Update

↓

Delete

↓

Restore (if supported)

Ensure every operation updates the data correctly.

---

# 6. Search Validation

Verify searching using:

- Complete keyword
- Partial keyword
- Uppercase
- Lowercase
- Numbers
- Special characters
- Emoji
- Leading spaces
- Trailing spaces
- SQL keywords (validation only)

---

# 7. Filter Validation

Verify:

- Single filter
- Multiple filters
- Reset filter
- Empty filter
- Invalid filter

---

# 8. Sorting Validation

Verify sorting for:

- Ascending
- Descending
- Alphabet
- Number
- Date
- Empty values
- Mixed values

---

# 9. Pagination Validation

Verify:

- First page
- Last page
- Next page
- Previous page
- Different page sizes
- Empty pages

---

# 10. File Upload Validation

Whenever file upload exists, verify:

## File Type

- PDF
- JPG
- PNG
- JPEG
- DOCX
- XLSX
- ZIP
- EXE
- HTML
- JS

## File Size

- Empty file
- Very small file
- Maximum allowed size
- Above maximum size

## File Name

Verify filenames containing:

- Spaces
- Unicode
- Long filenames
- Special characters

## File Integrity

Verify:

- Corrupted files
- Renamed extensions
- Invalid MIME type
- Empty document

Whenever possible, use the prepared testing files located inside the local testing assets folder provided by the user instead of generating new files.

---

# 11. Network Validation

Inspect browser DevTools.

Review:

- Failed requests
- Request payload
- Response payload
- HTTP Status
- Timeout
- Retry
- Duplicate requests

---

# 12. API Validation

Whenever API responses are visible, verify:

- Status Code
- Error Message
- Success Message
- Response Structure
- Missing Properties
- Unexpected Properties
- Response Time

---

# 13. UI Validation

Inspect:

- Alignment
- Overflow
- Responsive layout
- Broken UI
- Incorrect icons
- Incorrect colors
- Placeholder
- Tooltip
- Loading indicator
- Spinner
- Skeleton loader

---

# 14. Responsive Testing

Verify:

Desktop

Laptop

Tablet

Mobile

Landscape

Portrait

---

# 15. Session Validation

Verify:

- Refresh page
- Logout
- Session expiration
- Browser Back
- Browser Forward
- Multiple Tabs

---

# 16. Role & Permission Validation

Whenever RBAC exists, verify:

- Administrator
- Manager
- Staff
- Guest
- Unauthorized User

Ensure every role only accesses permitted functionality.

---

# 17. User Behavior Testing

Simulate unexpected user actions.

Examples:

- Double click
- Triple click
- Spam click
- Rapid typing
- Refresh during submission
- Browser Back
- ESC
- ENTER
- Drag & Drop
- Cancel upload
- Close modal while processing

---

# 18. Error Handling

Verify that failures are handled correctly.

Check:

- Error messages
- Validation messages
- Loading behavior
- Infinite loading
- Blank pages
- White screen
- Unexpected crashes

---

# 19. Basic Security Validation

Without performing penetration testing, verify input validation against common malicious payloads.

Examples:

- HTML tags
- Script tags
- SQL keywords
- Directory traversal patterns
- Quote characters
- Unexpected symbols

The objective is only to verify application validation.

---

# 20. Data Integrity Validation

Ensure consistency across:

UI

↓

API

↓

Database

↓

Audit Log

↓

History

↓

Export

↓

Notification

---

# 21. Workflow Validation

Verify every business workflow.

Example

Draft

↓

Submit

↓

Approve

↓

Reject

↓

Cancel

↓

Reopen

Ensure every state transition is valid.

---

# 22. UX Observation

Besides identifying defects, observe opportunities to improve user experience.

Examples:

- Confusing workflow
- Excessive number of clicks
- Unclear labels
- Poor validation messages
- Inconsistent buttons
- Missing feedback
- Difficult navigation

Whenever a UX improvement is identified, provide the recommendation using:

Baiknya ...

...

(Jika memungkinkan)

Do not classify UX suggestions as defects unless they violate an approved requirement.

---

# General Rules

During exploratory testing, always prioritize:

1. Functional correctness
2. Business rule validation
3. Edge cases
4. User behavior
5. Data integrity
6. Security validation
7. Usability improvements

The objective is to maximize test coverage while producing actionable and reproducible findings for developers.

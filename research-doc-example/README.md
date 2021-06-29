# OCR API

## Purpose

The purpose of this document is to consolidate the various options afforded to us when creating a OCR system to support document content search.

## Background

Customers often require the ability to locate specific documents based on the text that they
contain. When utilizing this type of search, we need to expose the ability to search specific
categories of documents and display the appropriate results. Locating specific text within documents
provides the users with quicker turn-around times for document creation and reduces downloading
multiple documents.

This decision will impact the results of the aforementioned needs of our customers. We should move
forward with a cost-effective and robust tool that will reduce our overhead while supplying the user
with a highly accurate and extensible tool to search documents

## Relevant data

- What is the budget for this feature?
- What level of accuracy are we willing to live with?
- What is our SLA for successful translation?
- What is our minimum list of file types we must support?
- How many documents/requests will we transcribe per Month?

## Options considered

|      | Tesseract Gem                                    | Google Cloud Vision                   | CloudMersive                                       |
| ---- | ------------------------------------------------ | ------------------------------------- | -------------------------------------------------- |
| url  | https://github.com/tesseract-ocr/tessdoc         | https://cloud.google.com/vision       | https://cloudmersive.com/ocr-api                   |
| Pros | Native Gem Support                               | Auto Detects Language                 | Auto Detects Language                              |
|      | No File Size Limitations (within Ruby’ s Limits) | All Text (Blob)\*\* or Text By Region | All Text (Blob)\*\*                                |
|      | OpenSource                                       | Additional recognition libraries      | Subscription Pricing                               |
|      | Multi Language Support\*                         | PDF, TIFF, “common image files”       | JPEG, PNG, “common image files”                    |
|      | All Text (Blob)\*\*                              | Photos or Scans                       | JSON Response type                                 |
|      |                                                  | HIGH fidelity/fault-tolerance         | Optional fidelity/fault-tolerance                  |
| Cons | Cannot compete with fault tolerance              | 30 requests/second                    | Higher fault tolerances consumes more API requests |
|      | Maintenance/Support                              | Could get pricey if used frequentl    | 2 requests/second                                  |
|      | Extra Debian Depedencies                         | PDF page limitation - 2000 pages      | File Size Limits                                   |
| Cost | Free-ish                                         | ~ \$13.50 / 10,000 document           | ~ \$20 / 10,000 documents                          |

### Action items

- [x] @Rebecca initial research into available options
- [ ] @Bob address above concerns with stakeholders
- [ ] @Dev move forward with desgin doc for chosen option

```text
##  Recommendations:

Option 1, The upfront monetary savings is not offset by the heavy lift to integrate the most basic
functionality and will be fairly limited in the long run with the greatest support required by
internal development staff.

Option 2, The lowest bar of entry into an extensive ecosystem and deep integration with our GCP
infrastructure could be a huge benefit. It is the fastest, smartest, and could be cost-effective if
the request usage spread is large.

Option 3, Higher cost over a longer period of time but this cost could be reduced by moving up to
the Enterprise level service plans.
```

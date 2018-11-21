# BRE information sources

## All sources table

| Service name | Type | Comment | Has disable flag |
| -- | -- | -- | -- |
| WebInvoice (customer info) |Internal - core | | No - but we are returning empty response for new customers |
| Balances Service | Internal - Core| |  Yes - empty response is returned |
| CCS live | Internal | Not available after migration | Exception - can be converted to flag |
| CCS backup | Internal|BRE has copy of data and job that fetches them Probably cannot be migrated | Same as live, exception but can be converted to flag and empty response can be returned |
| LIS (customer information) | Internal - LIS	| | No - empty response is returned for new customers  |
| LIS (customer stats) | Internal - LIS	| | No - empty response is returned for new customers  |
| Purchase Behaviour Service |Internal - BRE | |  |
| Scoring DB | Internal - BRE | | |
| Bisnode FI (Soliditet) | External |
| SAT | External |
| VRK | External | SSL certificate installation needed |

## Existing flags

### Finland

Disable Balance service
Disable Lis backup

## Collection system situation

- CCS FI - no collection, empty result is returned. Can be converted to flag, since currently we are using decision exception to skip check
- CCS DK - we are using decision exception to skip check, lack of data fails the decision.
- CCS SE - we are using decision exception to skip check, empty result with SuccessfulSearch = true skips check. The same used in Pre Credit SE
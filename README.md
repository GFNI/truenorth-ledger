# TrueNorth Ledger
![Tests](https://img.shields.io/badge/tests-passing-green) ![Coverage](https://img.shields.io/badge/coverage-100%25-green)
This repository serves as the **public ledger** for all random number generation (RNG) events conducted by TrueNorth.
## System Verification
The integrity of this ledger is backed by a comprehensive test suite ensuring the RNG process, cryptographic hashing, and GitHub integration work exactly as specified.
**Current Status**:
- **Tests Passing**: 123/123
- **Assertions**: 385
- **Last Verified**: 2025-11-30
## Purpose
We believe in absolute transparency and long-term accountability. Every time a draw is conducted on our platform, the cryptographic proofs and parameters are automatically committed to this repository. This ensures that:
1. **Independent Persistence**: This ledger exists outside of our internal databases. Even if TrueNorth ceases to exist, these results remain accessible and verifiable forever.
2. **History cannot be altered**: The Git commit history serves as a timestamped, tamper-evident log.
3. **Verification is public**: Anyone can audit our draws using the data stored here.
## How It Works
Our RNG system is "Provably Fair". It relies on **Drand**, a distributed randomness beacon. Drand does not supply the final random numbers directly; instead, it provides an unpredictable, publicly verifiable random string which we use as the core seed to generate the results.
For each draw, we combine:
1. **Drand Server Seed**: The randomness from a specific Drand round.
2. **Client Seed**: A timestamp generated at the moment of the request.
3. **Round ID**: A unique UUID for the draw.
4. **Static Salt**: A public constant unique to TrueNorth.
5. **Ticket Parameters**: Total tickets sold and max tickets.
These inputs are hashed using **HMAC-SHA256** and converted to a number using **Unbiased Rejection Sampling**.
## File Structure
Records are organized by date to ensure scalability:
```text
/YYYY
  /MM
    /DD
      /round-uuid.json
```
## Verifying a Draw
Each JSON file contains all the public parameters needed to verify the result.
### Example JSON
```json
{
    "round_id": "b564cfe5-97bc-4014-b630-4138ca914088",
    "draw_title": "£3,000",
    "competition_url": "https://rafflex.io/",
    "operator": {
        "name": "TrueNorth",
        "website": "https://truenorth.rafflex.io",
        "company_number": 10101010
    },
    "drand_server_seed": "8f4a2b1c9e7d3f5a6b8c0d2e4f6a8b0c2d4e6f8a0b2c4d6e8f0a2b4c6d8e0f2a",
    "drand_seed_hash": "a0652cb91fee10a77f8a4b31ba3a5b084c460531bd34114782a265eb6006e5e93a39cb5e1fc1eee21451e864d8908d88",
    "client_seed": "1764428274572",
    "static_salt": "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5z6a7b8c9d0e1f2",
    "tickets_sold": 4392,
    "max_tickets": 5000,
    "combined_hash": "db4268498d15a9971e39a65001c29ad3632dfd030d6e061b3659a975203824a3",
    "result": "4628",
    "external_round_id": "23874972",
    "processing_status": "completed",
    "created_at": "2025-11-29T14:57:54.000000Z",
    "revealed_at": "2025-11-29T14:58:05.000000Z"
}
```
### Field Descriptions
#### Draw Identification
| Field | Type | Description |
|-------|------|-------------|
| `round_id` | `string` | Unique UUID identifying this specific draw. Used as the filename and primary reference. |
| `draw_title` | `string` | Human-readable name or prize description for the draw. |
| `competition_url` | `string` | URL to the competition page where this draw was conducted. |
| `external_round_id` | `string` | The Drand beacon round number used for this draw. Can be verified at [drand.love](https://drand.love). |
#### Operator Information
| Field | Type | Description |
|-------|------|-------------|
| `operator.name` | `string` | Company or individual name conducting the draw. |
| `operator.website` | `string` | Official website of the operator. |
| `operator.company_number` | `integer` | Registered company number for regulatory compliance. |
#### Cryptographic Inputs
| Field | Type | Description |
|-------|------|-------------|
| `drand_server_seed` | `string` | The raw randomness value from the Drand beacon. This is the unpredictable entropy source used in the HMAC-SHA256 calculation. |
| `drand_seed_hash` | `string` | The Drand signature (BLS signature) proving the randomness came from the distributed beacon. Independently verifiable against the Drand network. |
| `client_seed` | `string` | Millisecond-precision timestamp generated server-side at draw initiation. Ensures uniqueness across draws. |
| `static_salt` | `string` | A public constant unique to TrueNorth, used across all draws. Proves the draw originated from TrueNorth and adds consistent entropy to the hash. |
#### Ticket Parameters
| Field | Type | Description |
|-------|------|-------------|
| `tickets_sold` | `integer` | Total number of tickets sold for this draw. Tickets are randomly allocated between 1 and `max_tickets`. |
| `max_tickets` | `integer` | Maximum ticket capacity for the draw. Used in the hash calculation for consistency. |
#### Result Data
| Field | Type | Description |
|-------|------|-------------|
| `combined_hash` | `string` | The HMAC-SHA256 hash produced by combining all cryptographic inputs. This is the deterministic proof that connects inputs to the result. |
| `result` | `string` | The winning ticket number, derived from `combined_hash` using unbiased rejection sampling. |
| `processing_status` | `string` | Status of the draw: `pending` (awaiting Drand beacon) or `completed` (result calculated). |
#### Timestamps
| Field | Type | Description |
|-------|------|-------------|
| `created_at` | `string` | ISO 8601 timestamp when the draw was initiated. |
| `revealed_at` | `string` | ISO 8601 timestamp when the Drand randomness was revealed and the result calculated. |
## Verification
To verify a draw result:
1. **Verify Drand**: Confirm `drand_server_seed` matches the Drand round at `external_round_id` using [drand.love](https://api.drand.sh/dbd506d6ef76e5f386f41c651dcb808c5bcbd75471cc4eafa3f4df7ad4e4c493/public/{external_round_id})
2. **Recreate the hash**: Combine inputs using HMAC-SHA256 with the formula:
   ```
   HMAC-SHA256(drand_server_seed, "client_seed:round_id:static_salt:tickets_sold:max_tickets")
   ```
3. **Verify the result**: Apply unbiased rejection sampling to confirm the `result` matches.
You can use our [Verification Tool](https://truenorth.rafflex.io/verify) or write your own script using the parameters found in these files.

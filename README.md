# TrueNorth Ledger

This repository serves as the **public ledger** for all random number generation (RNG) events conducted by TrueNorth.

## Purpose
We believe in absolute transparency and long-term accountability. Every time a draw is conducted on our platform, the cryptographic proofs and parameters are automatically committed to this repository. This ensures that:
1.  **Independent Persistence**: This ledger exists outside of our internal databases. Even if TrueNorth ceases to exist, these results remain accessible and verifiable forever.
2.  **History cannot be altered**: The Git commit history serves as a timestamped, tamper-evident log.
3.  **Verification is public**: Anyone can audit our draws using the data stored here.

## How It Works
Our RNG system is "Provably Fair". It relies on **Drand**, a distributed randomness beacon. Drand does not supply the final random numbers directly; instead, it provides an unpredictable, publicly verifiable random string which we use as the core seed to generate the results.

For each draw, we combine:
1.  **Server Seed**: The randomness from a specific Drand round.
2.  **Client Seed**: A timestamp generated at the moment of the request.
3.  **Round ID**: A unique UUID for the draw.
4.  **Static Salt**: A constant server-side secret.
5.  **Ticket Parameters**: Total tickets sold and max tickets.

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
    "competition_url": "https://truenorth.rafflex.io/competition/3000gbp-cash",
    "operator": {
        "name": "TrueNorth",
        "website": "https://truenorth.rafflex.io",
        "company_number": 10101010
    },
    "server_seed_hash": "a0652cb91fee10a77f8a4b31ba3a5b084c460531bd34114782a265eb6006e5e93a39cb5e1fc1eee21451e864d8908d88",
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

To verify, you can use our [Verification Tool](https://truenorth.rafflex.io/verify) or write your own script using the parameters found in these files.

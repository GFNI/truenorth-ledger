# TrueNorth Ledger

This repository serves as the **public ledger** for all random number generation (RNG) events conducted by TrueNorth.

## Purpose
We believe in absolute transparency and long-term accountability. Every time a draw is conducted on our platform, the cryptographic proofs and parameters are automatically committed to this repository. This ensures that:
1.  **Independent Persistence**: This ledger exists outside of our internal databases. Even if TrueNorth ceases to exist, these results remain accessible and verifiable forever.
2.  **History cannot be altered**: The Git commit history serves as a timestamped, tamper-evident log.
3.  **Verification is public**: Anyone can audit our draws using the data stored here.

## How It Works
Our RNG system is "Provably Fair". It relies on **Drand**, a distributed randomness beacon that generates unpredictable, publicly verifiable random numbers.

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
  "round_id": "550e8400-e29b-41d4-a716-446655440000",
  "external_round_id": "1234567",
  "server_seed": "8d9...",
  "client_seed": "1732837...",
  "tickets_sold": 150,
  "max_tickets": 200,
  "result": "42",
  "revealed_at": "2025-11-29T12:00:00.000000Z"
}
```

To verify, you can use our [Verification Tool](https://truenorth.rafflex.io/verify) or write your own script using the parameters found in these files.

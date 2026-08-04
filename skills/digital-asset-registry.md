---
name: Digitalasset
description: Use when building tokenization platforms, issuing digital assets on Canton, managing asset lifecycles (mint/burn/transfer), configuring compliance controls (allowlists/blocklists), or integrating with the Registry APIs. Reach for this skill when working with institutional-grade asset issuance, token standards, or multi-party asset workflows.
metadata:
    mintlify-proj: digitalasset
    version: "1.0"
---

# Digital Asset Registry Skill

## Product Summary

The **DA Registry** is a production-ready platform for issuing, managing, and monetizing institutional-grade digital assets on the Canton Network. It provides a complete asset lifecycle framework with built-in compliance controls, credential-based access management, and seamless integration with the Canton Token Standard (CIP-56). Agents use this skill to configure instruments, manage supply (mint/burn), execute transfers, and enforce regulatory controls.

**Key files and endpoints:**
- Registry dApp UI: https://registry.app.digitalasset.com/ (MainNet), https://registry.test.app.digitalasset.com/ (TestNet), https://registry.dev.app.digitalasset.com/ (DevNet)
- Operator Backend API: https://api.utilities.digitalasset.com (MainNet), https://api.utilities.digitalasset-staging.com (TestNet), https://api.utilities.digitalasset-dev.com (DevNet)
- DAR packages: Download from Releases page; install `utility-registry-app-v0` on your node
- Primary docs: https://docs.digitalasset.com/registry/overview

## When to Use

Reach for this skill when:

- **Setting up a tokenization platform:** Onboarding providers, registrars, and configuring infrastructure
- **Issuing digital assets:** Creating instrument configurations, defining credential requirements, minting tokens
- **Managing asset lifecycles:** Burning tokens, executing transfers, handling redemptions
- **Implementing compliance:** Configuring allowlists (credential-based access), blocklists (sanction screening), or delegating compliance to third parties
- **Integrating with Canton:** Building apps that transact Registry holdings, implementing token-standard workflows, or reading off-ledger metadata
- **Troubleshooting asset operations:** Diagnosing failed transfers, mint/burn rejections, or credential mismatches
- **Recovering assets:** Helping holders migrate holdings to new nodes after infrastructure failure

## Quick Reference

### Core Roles & Hierarchy

| Role | Responsibility | Scope |
|------|---|---|
| **Provider** | Operates Registry infrastructure, onboards registrars | Manages provider-level access and configuration |
| **Registrar** | Creates instruments, manages asset lifecycle | Administers one or more instruments |
| **Holder** | Owns asset holdings | Holds, transfers, or redeems tokens |
| **Issuer** | Requests minting/burning | Initiates supply changes (subject to registrar approval) |

### Asset Model

- **Instrument:** The asset definition (ISIN, CUSIP, credential rules). Created once per asset type.
- **Holding:** Individual ownership record. One instrument can have many holdings across different parties.

### Core Workflows

| Workflow | Initiator | Approver | Outcome |
|----------|-----------|----------|---------|
| **Mint** | Issuer (if credentials met) | Registrar | New holdings created |
| **Burn** | Issuer (if credentials met) | Registrar | Holdings permanently removed |
| **Transfer (Offer/Accept)** | Sender | Receiver | Ownership moves after explicit acceptance |
| **Transfer (Pre-approved)** | Sender | None (auto-executes) | Instant settlement if pre-approval exists |

### API Base URLs

```
MainNet:  https://api.utilities.digitalasset.com
TestNet:  https://api.utilities.digitalasset-staging.com
DevNet:   https://api.utilities.digitalasset-dev.com
```

### Credential Requirements

Define separately for each instrument:

- **Holder requirements:** Who may hold, receive, or transfer the asset
- **Issuer requirements:** Who may request minting or burning

Example configurations:
- **Stablecoin:** No holder requirement; issuer must have `canMint` credential
- **Money Market Fund:** Both holder and issuer must have `onboarded` credential
- **Regulated Bond:** Compliance provider issues `canHold` and `canMint` credentials

## Decision Guidance

### When to Use Offer/Accept vs Pre-Approved Transfer

| Scenario | Use Offer/Accept | Use Pre-Approval |
|----------|------------------|------------------|
| One-off transfers requiring explicit consent | ✓ | |
| High-frequency automated settlement | | ✓ |
| Receiver wants to control inbound transfers | ✓ | |
| Sender needs instant settlement | | ✓ |
| Receiver pre-authorizes specific instruments | | ✓ |

### When to Delegate Compliance vs Self-Manage

| Approach | When to Use | Trade-offs |
|----------|------------|-----------|
| **Self-managed allowlist** | Small, controlled investor base; issuer has KYC capability | Issuer bears full compliance burden |
| **Delegated to compliance provider** | Large investor base; third-party KYC/AML integration needed | Issuer depends on external provider; requires credential coordination |
| **Blocklist only** | Sanction screening is primary concern; no holder restrictions | Allows any party to hold; only blocks bad actors |

### When to Enable Blocklist Check

| Condition | Enable? | Reason |
|-----------|---------|--------|
| Stablecoin with sanction screening | ✓ | Regulatory requirement |
| Private fund with investor allowlist only | | Allowlist alone sufficient |
| Asset with both allowlist and blocklist | ✓ | Defense-in-depth compliance |
| Legacy system compatibility required | | Blocklist incompatible with pre-0.13 models |

## Workflow

### Typical Asset Issuance Flow

1. **Verify prerequisites:**
   - Commercial agreement with Digital Asset executed
   - Canton validator deployed and onboarded (or using CIP-103 wallet)
   - Reward sharing configured per commercial terms
   - Access to Registry dApp UI via CIP-103 compliant wallet

2. **Request Provider Service:**
   - Navigate to Registry dApp UI
   - Submit Provider Service request
   - Wait for approval (auto-approved on DevNet)

3. **Configure Provider (if needed):**
   - Define credential requirements for registrar onboarding
   - Issue credentials to parties requesting registrar status

4. **Onboard Registrar:**
   - Registrar party requests Registrar Service
   - Verify credential requirements are met
   - Accept or reject request

5. **Create Instrument Configuration:**
   - Define instrument identifier (e.g., `BOND`, `USDX`)
   - Add external identifiers (ISIN, CUSIP)
   - Define holder credential requirements
   - Define issuer credential requirements
   - Enable blocklist check if needed (create empty blocklist first)

6. **Issue Credentials (if required):**
   - Create credential contracts for holders and issuers
   - Ensure issuer holds the credential (simplifies management)
   - Verify claims match instrument configuration

7. **Mint Initial Supply:**
   - Issuer creates Mint request (amount, instrument, registrar)
   - Registry validates issuer credentials
   - Registrar reviews and accepts
   - Holdings created for issuer

8. **Execute Transfers:**
   - Sender creates Transfer offer or uses pre-approval
   - Registry validates holder credentials for both parties
   - Receiver accepts (offer/accept) or transfer executes (pre-approved)

9. **Manage Compliance:**
   - Add/remove parties from blocklist as needed
   - Update allowlist credentials for new investors
   - Monitor transfer activity for compliance violations

### Typical API Integration Flow

1. **Retrieve operator party ID:**
   ```
   GET /api/utilities/v0/operator
   ```

2. **Fetch instrument metadata:**
   ```
   GET /api/token-standard/v0/registrars/{registrarId}/registry/metadata/v1/info
   GET /api/token-standard/v0/registrars/{registrarId}/registry/metadata/v1/instruments
   ```

3. **Create transfer instruction:**
   ```
   POST /api/token-standard/v0/registrars/{registrarId}/registry/transfer-instruction/v1/transfer-factory
   ```

4. **Accept/reject/withdraw transfer:**
   ```
   POST /api/token-standard/v0/registrars/{registrarId}/registry/transfer-instruction/v1-choice-contexts/accept
   POST /api/token-standard/v0/registrars/{registrarId}/registry/transfer-instruction/v1-choice-contexts/reject
   POST /api/token-standard/v0/registrars/{registrarId}/registry/transfer-instruction/v1-choice-contexts/withdraw
   ```

5. **Create allocation (pre-approval):**
   ```
   POST /api/token-standard/v0/registrars/{registrarId}/registry/allocation-instruction/v1/allocation-factory
   ```

## Common Gotchas

- **Blocklist incompatibility:** Enabling blocklist check on an instrument makes it incompatible with Registry models before version 0.13. Create an empty blocklist before enabling the check, or transfers/mints/burns will fail.

- **Credential mismatch:** If a party lacks required credentials, the transaction fails at the ledger level. Verify credentials exist and match the instrument configuration exactly (issuer, property, value).

- **Holdings locked during pending operations:** When a mint/burn/transfer is pending, the affected holdings are locked and cannot be used in other operations. Cancel or wait for approval to release them.

- **Pre-approval scope:** A Transfer Preapproval can specify up to 10 instrument IDs or apply to all instruments from a registrar. Verify the preapproval covers the intended instrument before initiating transfer.

- **Registrar approval required:** Mint and burn workflows require explicit registrar acceptance. Requesters cannot unilaterally create supply; they can only cancel pending requests.

- **Transfer rule missing:** Transfers fail if no `TransferRule` contract exists for the instrument. Instrument admins must instantiate this via the dApp UI.

- **Credential issuer must hold credential:** For allowlist management, the credential issuer should also hold the credential. This avoids requiring the target party to separately accept it.

- **Blocklist sharding cost:** Blocklist transactions disclose additional contracts. The blocklist is sharded by party prefix to bound transaction costs.

- **Asset recovery requires issuer validation:** Holders cannot initiate asset recovery alone. The issuer must verify identity and approve the recovery request.

- **CIP-103 wallet requirement:** The Registry dApp UI requires a CIP-103 compliant wallet. If running your own validator, deploy the Wallet Gateway to connect enterprise signing providers (Fireblocks, BitGo, Blockdaemon, Dfns).

## Verification Checklist

Before submitting asset issuance or transfer workflows:

- [ ] Instrument Configuration exists and is disclosed on-ledger
- [ ] All required credentials are issued and held by the acting parties
- [ ] Credential claims (issuer, property, value) match the instrument configuration exactly
- [ ] Blocklist is instantiated if blocklist check is enabled on the instrument
- [ ] TransferRule contract exists if transfers are enabled
- [ ] AllocationFactory contract exists if using token-standard transfers
- [ ] Registrar party is onboarded and has accepted the Registrar Service
- [ ] Provider Configuration (if any) has been created and requirements are met
- [ ] For pre-approved transfers: Transfer Preapproval exists and covers the target instrument
- [ ] For mint/burn: Issuer has required credentials and registrar is available to approve
- [ ] For transfers: Both sender and receiver meet holder credential requirements
- [ ] Blocklist does not contain either party (if blocklist check enabled)
- [ ] Holdings are not locked by pending operations
- [ ] API calls use correct environment endpoint (MainNet/TestNet/DevNet)

## Resources

**Comprehensive page navigation:** https://docs.digitalasset.com/llms.txt

**Critical documentation pages:**
- [Registry Primer & Architecture](https://docs.digitalasset.com/registry/get-started/registry-primer) — Understand the platform pillars and role model
- [Quickstart & Prerequisites](https://docs.digitalasset.com/registry/get-started/quickstart) — Get Provider Service set up
- [Guides & Integration](https://docs.digitalasset.com/registry/guides/provider-configuration) — Step-by-step workflows for all operations

---

> For additional documentation and navigation, see: https://docs.digitalasset.com/llms.txt
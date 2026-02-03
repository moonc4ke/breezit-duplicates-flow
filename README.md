## Complete Email/SMS Processing Flow

```
                                 Email/SMS Webhook
                                        │
                                        ▼
                        ┌───────────────────────────────┐
                        │    Is from Old Customer?      │
                        │   (check oldCustomers.list)   │
                        └───────────────────────────────┘
                                        │
                           ┌────────────┴────────────┐
                           │                         │
                          YES                        NO
                           │                         │
                           ▼                         ▼
                ┌─────────────────────────────────┐   ┌─────────────────────────┐
                │ IF templateResponse.enabled     │   │  Is from Lead Platform? │
                │   → Send template email to      │   └─────────────────────────┘
                │     customer                    │              │
                │                                 │         ┌────┴────────────────────────┐
                │ IF triggers.oldCustomerContact  │         │                             │
                │    .enabled                     │        YES                            NO
                │   → For each recipient:         │         │                             │
                │     IF receiveSms: Send SMS     │         │                             │
                │     IF receiveEmail: Send Email │         │                             │
                │                                 │         │                             │
                │ • Charge 0.04 ct                │         │                             │
                │ • STOP                          │         │                             │
                └─────────────────────────────────┘         ▼                             │
                                                      ┌───────────────┐                   │
                                                      │   Agent 0     │                   │
                                                      │ (extract ALL  │                   │
                                                      │ customer      │                   │
                                                      │ emails array) │                   │
                                                      └───────────────┘                   │
                                                         │                                │
                                                         ▼                                │
                                   ┌────────────────────────┐                             │
                                   │ Check if ANY extracted │                             │
                                   │ email has existing     │                             │
                                   │ booking                │                             │
                                   └────────────────────────┘                             │
                                                │                                         │
                                         ┌──────┴──────┐                                  │
                                         │             │                                  │
                                        YES           NO                                  │
                                         │             │                                  │
                                         ▼             │                                  │
                              ┌────────────────┐       │                                  │
                              │  AI enabled?   │       │                                  │
                              └────────────────┘       │                                  │
                                      │                │                                  │
                                 ┌────┴────┐           │                                  │
                                 │         │           │                                  │
                                YES       NO           │                                  │
                                 │         │           │                                  │
                                 │         ▼           │                                  │
                                 │  ┌───────────────────────────────┐                     │
                                 │  │ IF triggers                   │                     │
                                 │  │    .aiDisabledBookingMessage  │                     │
                                 │  │    .enabled                   │                     │
                                 │  │   → For each recipient:       │                     │
                                 │  │     IF receiveSms: Send SMS   │                     │
                                 │  │     IF receiveEmail: Send Email│                    │
                                 │  │ • Charge 0.02 ct per message  │                     │
                                 │  │ • STOP (manual response)      │                     │
                                 │  └───────────────────────────────┘                     │
                                 │                     │                                  │
                                 ▼                     ▼                                  ▼
                    ┌──────────────────┐    ┌──────────────────┐      ┌────────────────────────────┐
                    │ Get latest       │    │                  │      │ Check if sender email has  │
                    │ booking +        │    │                  │      │ existing booking           │
                    │ conversation     │    │                  │      └────────────────────────────┘
                    │ history          │    │                  │                     │
                    └──────────────────┘    │                  │              ┌──────┴───────────────────────────────┐
                             │              │                  │              │                                      │
                             ▼              │                  │             YES                                     NO
                    ┌──────────────────┐    │                  │              │                                      │
                    │   Agent 1        │    │   Agent 1        │              ▼                                      │
                    │   (Option 2)     │    │   (Option 1)     │   ┌────────────────┐                                │
                    │   With history   │    │   No history     │   │  AI enabled?   │                                │
                    └──────────────────┘    └──────────────────┘   └────────────────┘                                │
                             │                       │                    │                                          │
                             │                       │               ┌────┴────┐                                     │
                             │                       │               │         │                                     │
                             │                       │              YES       NO                                     │
                             │                       │               │         │                                     │
                             │                       │               │         ▼                                     │
                             │                       │               │  ┌───────────────────────────────┐            │
                             │                       │               │  │ IF triggers                   │            │
                             │                       │               │  │    .aiDisabledBookingMessage  │            │
                             │                       │               │  │    .enabled                   │            │
                             │                       │               │  │   → For each recipient:       │            │
                             │                       │               │  │     IF receiveSms: Send SMS   │            │
                             │                       │               │  │     IF receiveEmail: Send Email│           │
                             │                       │               │  │ • Charge 0.02 ct per message  │            │
                             │                       │               │  │ • STOP (manual response)      │            │
                             │                       │               │  └───────────────────────────────┘            │
                             │                       │               │                                               │
                             │                       │               ▼                                               │
                             │                       │    ┌──────────────────┐                                       │
                             │                       │    │ Get booking +    │                                       │
                             │                       │    │ conversation     │                                       │
                             │                       │    │ history          │                                       │
                             │                       │    └──────────────────┘                                       │
                             │                       │              │                                                │
                             │                       │              ▼                                                ▼
                             │                       │    ┌──────────────────┐                               ┌──────────────────┐
                             │                       │    │   Agent 1        │                               │   Agent 1        │
                             │                       │    │   (Option 2)     │                               │   (Option 1)     │
                             │                       │    │   With history   │                               │   No history     │
                             │                       │    └──────────────────┘                               └──────────────────┘
                             │                       │              │                                                │
                             └───────────────────────┴──────────────┴────────────────────────────────────────────────┘
                                                                    │
                                                                    ▼
                                                    ┌───────────────────────────────┐
                                                    │        Agent 1 Decision       │
                                                    └───────────────────────────────┘
                                                                    │
                                   ┌────────────────────────────────┼────────────────────────────────┐
                                   │                                │                                │
                                   ▼                                ▼                                ▼
                            create_booking                     edit_booking                   duplicate_inquiry
                            no_email                                                         (Option 2 only)
                                   │                                │                                │
                                   │                                │                                ▼
                                   │                                │              ┌─────────────────────────────┐
                                   │                                │              │ Add "Breezit AI:            │
                                   │                                │              │ Duplicate Inquiry" label    │
                                   │                                │              │ (keep in INBOX)             │
                                   │                                │              │ • STOP                      │
                                   │                                │              └─────────────────────────────┘
                                   │                                │
                                   └────────────────┬───────────────┘
                                                    │
                                                    ▼
                                    ┌───────────────────────────────┐
                                    │     Agent 2 (email_extractor) │
                                    │        ← ALWAYS RUNS          │
                                    └───────────────────────────────┘
                                                    │
                                                    ▼
                                    ┌───────────────────────────────┐
                                    │   Agent 3 (create_email)  OR  │
                                    │   Agent 4 (edit_or_reply)     │
                                    └───────────────────────────────┘
                                                    │
                                                    ▼
                                             Send Email Response
```

# TripSplit — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

TripSplit is a Telegram bot that tracks shared trip expenses in group chats. It allows organizers to create trips, record expenses with custom splits, calculate live balances, and facilitate minimal settlement payments. All data is private to participants, with sensitive details accessible only via direct messages. The bot handles rounding, participant joins/leaves, and maintains an audit trail of all transactions.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- groups of friends
- travel companions
- small teams

## Success criteria

- All trip expenses are accurately tracked and reconciled to zero-sum balances
- Settlement payments are minimized and confirmed by participants
- Trip data remains private to participants with no exposure of sensitive details in group chats

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu with trip creation and overview options
- **/newtrip** (command, actor: user, command: /newtrip) — Create a new trip with currency selection and participant list
- **Add expense** (button, actor: user, callback: expense:add) — Record a new expense with payer, amount, description, and split type
  - inputs: payer, amount, description, split_type
  - outputs: expense_confirmation
- **/balance** (command, actor: user, command: /balance) — View anonymized balance summary in group chat or full details in DM
- **Settle payment** (button, actor: user, callback: settlement:init) — Record a confirmed payment between participants
  - inputs: payee, amount
  - outputs: settlement_confirmation
- **Manage participants** (button, actor: organizer, callback: trip:participants) — Add/remove participants or view join/leave history

## Flows

### Trip creation
_Trigger:_ /newtrip

1. Prompt for trip name
2. Select currency
3. Add participants from group members
4. Confirm trip creation

_Data touched:_ Trip

### Expense recording
_Trigger:_ Add expense button

1. Select payer
2. Enter amount
3. Add description
4. Choose split type (equal/custom)
5. Confirm split amounts
6. Post group summary and private details

_Data touched:_ Expense, Balance snapshot

### Balance viewing
_Trigger:_ /balance

1. Show anonymized group summary
2. Offer DM option for full details
3. Display per-participant balances and settlement plan

_Data touched:_ Balance snapshot

### Settlement recording
_Trigger:_ Settle payment button

1. Select payee
2. Confirm exact amount from balance plan
3. Request confirmation from payee
4. Update balances on confirmation

_Data touched:_ Settlement, Balance snapshot

### Participant management
_Trigger:_ Manage participants button

1. List active participants
2. Add new participant
3. Mark participant as left
4. Recalculate balances

_Data touched:_ Participant, Balance snapshot

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Trip** _(retention: persistent)_ — Trip metadata and participants
  - fields: title, currency, organizer, participants, expenses
- **Participant** _(retention: persistent)_ — User in the trip with balance tracking
  - fields: telegram_id, display_name, status, join_timestamp, leave_timestamp
- **Expense** _(retention: persistent)_ — Recorded expense with split details
  - fields: id, timestamp, payer, amount, description, shares, receipt
- **Balance snapshot** _(retention: session)_ — Current net positions of participants
  - fields: participant_id, net_amount
- **Settlement** _(retention: persistent)_ — Confirmed payment between participants
  - fields: payer, payee, amount, timestamp, confirmed

## Integrations

- **Telegram** (required) — Group chat and private message interactions
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Create/delete trips
- Edit participant lists
- Approve expense edits
- View full audit logs

## Notifications

- Balance updates when new expenses added
- Settlement confirmation requests
- Participant join/leave alerts

## Permissions & privacy

- Trip data visible only to participants
- Private messages required for sensitive details
- Organizer has full audit access

## Edge cases

- Rounding adjustments in equal splits
- Participants leaving mid-trip
- Disputed settlements requiring organizer resolution
- Custom splits with non-uniform amounts

## Required tests

- End-to-end expense recording with split validation
- Settlement confirmation workflow with dispute handling
- Participant removal and balance recalculation
- Audit trail persistence after trip deletion

## Assumptions

- Default currency based on organizer's locale
- Rounding remainder assigned to payer by default
- Organizer has final authority on disputes
- Group chat summaries remain non-sensitive

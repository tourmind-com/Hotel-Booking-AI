# Hotel Booking AI Skill

TourMind's end-to-end hotel booking skill for AI clients. Public queries require no login; creating or operating an order uses AgentAuth `user_key`.

## Features

- Resolves cities, hotels, landmarks, stations, and other POIs without inventing coordinates.
- Verifies up to 20 hotel candidates against live room products and selects the five best matches.
- Returns consistent hotel cards with images, distance, live prices, cancellation terms, tax status, and evidence-based match reasons.
- Returns hotel details, room images, beds, meals, live quotes, and cancellation terms together.
- Rechecks price and availability before booking.
- Creates bookings after collecting the guest's legal name and required contact email.
- Queries and cancels existing bookings.
- Starts Stripe, WeChat Pay, or Alipay payments and discloses Stripe's 3.5% processing fee when applicable.

## Installable contents

```
├── .gitignore
├── LICENSE
├── README.md
├── SKILL.md
├── references/
│   └── parameter_guide.md
```

Evaluation fixtures, test reports, review translations, and development-only validators are intentionally excluded from the installable release.

## Installation

Install from ClawHub:

```bash
openclaw skills install @tourmind/hotel-booking-ai
```

Or clone this repository into the OpenClaw skills directory:

```bash
mkdir -p ~/.openclaw/skills
git clone https://github.com/tourmind-com/Hotel-Booking-AI.git ~/.openclaw/skills/hotel-booking-ai
openclaw gateway restart
```

Hotel search, details, live-rate queries and availability checks work without authentication. When the user is ready to create or operate an order, sign in with Google at `https://auth.journione.ai`, copy the `user_key`, save it as `user_key.txt` in the installed skill directory, and restrict its permissions:

```bash
chmod 600 ~/.openclaw/skills/hotel-booking-ai/user_key.txt
```

The key file is excluded by `.gitignore` and must never be committed. No local MCP server is required; the skill calls the TourMind Skill API directly over HTTP.

## API

**Base URL:** `https://api.tourmind.com`

| Endpoint | Purpose |
|---|---|
| `POST /skill/toc/check_skill_update` | Check whether the installed Skill has an update |
| `POST /skill/toc/search_location` | Resolve a region, POI, or hotel |
| `POST /skill/toc/search_hotels` | Search hotel candidates |
| `POST /skill/toc/get_hotel_detail` | Get hotel details and images |
| `POST /skill/toc/query_room_rates` | Get live rooms and rates |
| `POST /skill/toc/check_room_availability` | Recheck price and availability |
| `POST /skill/toc/create_booking` | Create a booking |
| `POST /skill/toc/query_booking` | Query a booking |
| `POST /skill/toc/cancel_booking` | Cancel a booking |
| `POST /skill/toc/pay_order` | Start payment |

`check_skill_update`, hotel search, static detail, room-rate query and availability check require no `user_key`. `create_booking`, `query_booking`, `cancel_booking` and `pay_order` require the key from `user_key.txt`. An already stored valid key may be included in `search_hotels` and `query_room_rates` solely to receive a read-only `web_url`; never prompt for it during public queries.

The authoritative Skill version is declared immediately below the title in `SKILL.md`. Send it as `current_version` only to `check_skill_update`: once when the Skill is first used in a new conversation, and again when an existing conversation resumes after at least 24 hours of inactivity.

## Example

```
User: Find a hotel in Tokyo for two adults from April 28 to April 30.

Assistant: TourMind returned 20 candidates. I verified live rooms and rates and selected the five best matches.

           1. Example Hotel
           [hotel hero image]
           Lowest matching live rate: JPY 18,000 per night; JPY 36,000 total
           Why it matches: low verified total in this set; fits two adults; immediately bookable inventory.

           I can also show the remaining candidates or the complete returned pool.

User: Show me the standard rooms at the second hotel.

Assistant: Here are the hotel's details, room images, and current matching quotes. Choose a room and I will perform a final price and availability check.

User: Book the standard king room.

Assistant: Please provide the guest's full legal name and a contact email. The email is required for booking status and cancellation notifications.

User: Alex Smith, guest@example.com

Assistant: Booking created. TourMind order number: TM20260428001. Choose Stripe, WeChat Pay, or Alipay.
```

## API reference

See [references/parameter_guide.md](references/parameter_guide.md) for request fields, POI resolution, candidate ranking, image mapping, taxes, and booking rules. Resolve city and region IDs through the live `search_location` endpoint instead of relying on hardcoded values.

## TourMind hotel booking ecosystem

Choose the package that matches the audience and connection model:

| Audience | Integration | Authentication model | Repository |
|---|---|---|---|
| Consumer / ToC | Direct HTTP Skill | Public search and availability; `user_key` only for order operations | **[Hotel Booking AI](https://github.com/tourmind-com/Hotel-Booking-AI)** |
| Business / ToB | Direct HTTP Skill | Skill Token required for every API call | [TourMind Booking Skill](https://github.com/tourmind-com/Tourmind-Booking-Skillss) |
| Consumer / ToC | MCP package + companion Skill | Public MCP connection; `user_key` only for order operations | [Hotel Booking AI MCP](https://github.com/tourmind-com/Hotel-Booking-AI-MCP) |
| Business / ToB | MCP package + companion Skill | Bearer-authenticated MCP connection | [TourMind Booking MCP](https://github.com/tourmind-com/Tourmind-Booking-MCP) |

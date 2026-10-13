# LLD Practice Platform (prototype)

Practice loop: **choose problem → design → submit → feedback → review → try again**, with attempt history and a diff against your previous attempt.
Python 3.10+, **no third-party dependencies**.

## Run
```bash
python app.py                      # http://localhost:8000
# optional AI review:
export ANTHROPIC_API_KEY=...       # optional; LLD_LLM_MODEL overrides the model
python app.py
```
Try the example designs in `examples/` (`parking_lot_weak.lld` then `parking_lot_good.lld`) to see the progress diff.

## Test
```bash
python -m unittest discover -s tests -t .
```
44 tests cover parsing (incl. malformed input), rubric rules (incl. alternative valid designs), the state machine, the service end-to-end, AI failure/garbage/prompt-steering, retry, crash recovery and the HTTP API.

## Design format (DSL)
```
interface PricingStrategy
  calculateFee(ticket: Ticket): Money
class HourlyPricing implements PricingStrategy
  ratePerHour: Money
  calculateFee(ticket: Ticket): Money
```
Unindented = declaration (`class | abstract class | interface | enum`, optional `extends` / `implements`); indented = field `name: Type` or method `name(args): Return`. A JSON format is also supported.

## API
`GET /api/problems` · `GET /api/problems/{id}` · `POST /api/attempts` · `PUT /api/attempts/{id}` · `POST /api/attempts/{id}/submit` · `POST /api/attempts/{id}/retry` · `GET /api/attempts/{id}` · `GET /api/history?problem_id=`

## Docs
`RESEARCH.md` (research note) · `DESIGN.md` (design note) · `AI_USAGE.md`

## Limitations
Heuristic alias matching; no method-body analysis; no auth; polling UI; SQLite file DB. See DESIGN.md.

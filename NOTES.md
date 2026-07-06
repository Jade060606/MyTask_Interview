# Notes

## 1. One fix explained: the double-booking bug

`dates_overlap()` only checked whether the *new* booking's start date fell
inside the *existing* booking's range (`start_b <= start_a <= end_b`). Any
new booking that started before the existing one but still overlapped it
(e.g. an existing booking Jan 10–15, and a new one Jan 5–12) slipped through,
because the new booking's start date wasn't inside the existing range.

Two inclusive date ranges overlap unless one ends entirely before the other
begins, so the fix replaces that condition with:

```python
return start_a <= end_b and start_b <= end_a
```

This catches overlaps in both directions (new range starting earlier,
starting later, fully containing, or fully contained by the existing one).

## 2. Failure case caught by the fix

Existing booking: Canon DSLR Camera, **Jan 10–15**.
Before the fix, a new booking for the same camera from **Jan 5–12** was
wrongly accepted (its start date isn't inside Jan 10–15, so the old check
missed the overlap). After the fix, this is correctly rejected with a 409.

## 3. AI use

I used an AI assistant to help spot the root cause of each bug faster (e.g.
confirming that the overlap check only handled one direction, and that
`(to_date - from_date).days` is exclusive of the end day) and to sanity-check
wording in this file and the commit messages.

I checked the AI's suggestions by not taking them on faith: I read the
`app.py` diff line by line myself, then wrote small standalone test scripts
that import the app's functions directly and assert on expected outputs
(overlap true/false cases, day counts for same-day vs multi-day rentals,
`is_bookable()` for the maintenance item), and I exercised the actual HTTP
endpoints with Flask's test client for the double-booking, same-day-price,
and maintenance-equipment scenarios before considering each fix done.

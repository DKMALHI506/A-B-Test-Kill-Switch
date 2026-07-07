# A/B Test Kill-Switch

An n8n workflow that watches a live A/B test and flags when the "losing" variant
is actually losing you money — before someone notices three weeks later and asks
why nobody said anything.

## Why I built this

Most A/B test dashboards just show you percentages. They don't tell you two things
that actually matter: whether the difference is real or just noise, and what it's
costing you to keep running the test while you find out.

There's also a mistake almost everyone makes without realizing it: checking your
test results every day and stopping as soon as it "looks significant." That's
called the peeking problem, and it makes you think something is significant when
it isn't, just because you looked too many times. I wanted to actually build
something that accounts for that instead of just writing "p < 0.05" and calling
it a day.

## What it does

1. Pulls the current numbers for Variant A (control) and Variant B (the test)
2. Checks if B is significantly worse than A — using a threshold that gets
   stricter the longer the test has been running, so early lucky/unlucky streaks
   don't trigger a false alarm
3. Works out roughly how much money is being lost by letting the worse variant
   keep running
4. If it's bad enough, sends a Slack alert and (in the full version) pauses the
   experiment after someone approves it
5. If it's borderline, just sends a heads-up so a human can decide
6. If everything's fine, it logs that too and moves on

## How the "kill" decision works

- Low: nothing worth mentioning
- Medium: B is trending worse but not proven yet — alert only, no action
- High: statistically real, and costing real money — alert + pause step

The statistical part uses a simplified version of what's called a Pocock boundary.
The short version: instead of always requiring the classic 95% confidence bar,
the bar goes up a bit each day the test has been running, since you're allowed to
be wrong more the more chances you give yourself to be wrong.

## Setup

You'll need n8n running (I ran it locally through Docker) and a Slack workspace
to receive alerts.

1. Import `ab-test-kill-switch.json` into n8n
2. Connect your Slack credential on both Slack nodes
3. Set the channel ID you want alerts sent to
4. For the "Pause Experiment" step, point it at whatever your real pause/stop
   endpoint would be — I used a free webhook.site URL just to see a request go
   out somewhere real
5. Run it manually to test, then put it on a schedule once it works

The test data in this version is simulated (Variant B is deliberately made worse
so there's something to catch) — swapping that for a real experiment platform's
API would be the natural next step.

## What I'd add if I kept going

- Pull real experiment data instead of simulating it
- A proper sequential testing library instead of my simplified boundary
- A record of every day's numbers instead of just the latest snapshot, so you
  can see the trend over time, not just where it ended up

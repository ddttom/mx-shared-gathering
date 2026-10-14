# MX-effect eval: case set and run

Does MX metadata change what an agent decides? This folder holds the evidence for that test.

- `mx-effect.cases.json` is the case set. Each case is presented twice, once with the document's MX metadata and once without it. The file carries its own preregistration: the hypotheses, the lanes and the run command, recorded before the first run.
- `2026-10-08T19-19-32-946Z-mx-effect.run.json` is the run that answered it. It holds every decision on every lane, the repeat measurements, and the aggregates.

The result is published whichever way it fell. To reproduce it, run the preregistered command against the case set with the same models, then compare your aggregates with the ones in the run file.

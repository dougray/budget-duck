# Budget Duck

A free, open-source walking duck robot you can build for about **$38–$93** in parts, with an
inductive charging nest, using a 3D printer and basic soldering.

It starts from [Open Duck Mini v2](https://github.com/apirrone/Open_Duck_Mini), a ~$430 open
replica of Disney's BDX droid, and asks how cheap it can get. The full analysis is in
[COST-DOWN.md](COST-DOWN.md).

| Option | What it is | Est. parts incl. nest |
|---|---|---|
| A. Duckling | ~22 cm, 10 leg hobby servos with a pot-wiper feedback mod, ESP32-S3 | ≈ $93 |
| B. Smart-servo Duckling | A with Feetech SC09 bus servos | ≈ $165 |
| C. Full-size, trimmed | Open Duck Mini with fewer servos and a cheap IMU | ≈ $230 |
| D. Duck-shaped Otto | [Otto DIY](https://www.ottodiy.com/) 4-servo biped in a duck shell | ≈ $38 |

Prices are estimates from October 2026 listings; check them before ordering.

## Status

Design study only. Nothing here has been built or tested yet. CAD, firmware and print files
will land in `duckling/` and `otto-duck/`.

## Licence

Apache-2.0 for the Open Duck Mini-derived parts, CC-BY-SA-4.0 for the Otto-derived parts and the
documentation. See [LICENSE.md](LICENSE.md) and [NOTICE](NOTICE).

## Contributing

Enable the hooks once per clone (they run [gitleaks](https://github.com/gitleaks/gitleaks)):

```bash
git config core.hooksPath .githooks
```

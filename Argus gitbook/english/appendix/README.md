# Appendix

Supplementary references for quick lookup.

---

## Pages in this section

| Page | Topic |
|---|---|
| [Glossary](glossary.md) | Technical terms |
| [Path reference](paths.md) | Paths of all files and binaries |
| [License](license.md) | The GPL / proprietary split |
| [Tests](testing.md) | The automated test suite |
| [Test report](test-report.md) | Results of the latest full campaign |

---

## Command quick reference

```bash
# Build and install
make
sudo ./install.sh

# Daily use
argus-cli status
argus-cli count
argus-cli list --external
argus-cli sensitive
argus-cli verify

# Retention and mode
argus-cli mode
argus-cli retention

# Security
sudo argus-cli release
sudo argus-cli remove

# Tests
sudo tests/run_all.sh
```

# Repository maintenance

Author: [id-ex](https://github.com/id-ex)

Repository: https://github.com/id-ex/rp2040-zero-adxl345-usb

Original project files are MIT licensed; see LICENSE. Third-party work is not covered by this grant.

## Publication scope

- English and Russian READMEs, LICENSE and portable `config/adxl.cfg`.
- Two CAD renders and three build photographs.
- Only these 3D files: `rp2040z+adxl345-case.step` and `rp2040z+adxl345-cover.step`.
- No FreeCAD assembly, external component models, backups or host-specific configuration.

## Before pushing updates

Check photos for personal details and location metadata. Keep the serial ID as a placeholder in the public configuration. Do not publish machine-specific calibration values as universal defaults.

```bash
git status --short
git diff --check
git add README.md README.ru.md PUBLISHING.md LICENSE .gitignore config/ \
  3drender1.png 3drender2.png photo1.jpg photo2.jpg photo3.jpg \
  rp2040z+adxl345-case.step rp2040z+adxl345-cover.step
git diff --cached --stat
git diff --cached --check
git commit -m "Update project files"
git push origin main
```

Do not force-add ignored files.

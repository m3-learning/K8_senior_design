# K8_senior_design

## Pre Commit
This project utilizes pre-commit which ensures proper formatting convention.

Initialize pre-commit:
```
pip install pre-commit
```

Install and run pre-commit on all files
```
pre-commit install
pre-commit run --all-files
```

Stage and Apply pre-commit changes
```
git add .
git commit -m "Lint files with pre-commit"
```

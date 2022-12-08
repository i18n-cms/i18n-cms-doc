---
sidebar_position: 1
---

# Integrate a standalone i18n repository into your app

## Add i18n repository as a submodule to your app codebase

For example:

The main repository of this app: https://github.com/i18n-cms/i18n-cms

The i18n repository of this app: https://github.com/i18n-cms/i18n-cms-locales


```cmd title="Add submodule by running this in the main repository"
git submodule add https://github.com/i18n-cms/i18n-cms-locales public/locales
```

Learn more about [git submodule](https://git-scm.com/book/en/v2/Git-Tools-Submodules)



## Deploy by Github action 
### aws s3

[Example](https://github.com/i18n-cms/i18n-cms-locales/blob/main/.github/workflows/aws-s3.yml) (Using [S3-sync](https://github.com/marketplace/actions/s3-sync))<br/>
```yml
name: Upload to aws s3

on:
  push:
    branches:
    - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@master
    - uses: jakejarvis/s3-sync-action@master
      with:
        args: --acl public-read --follow-symlinks --delete --exclude '.git/*' --exclude '.github/*' --exclude 'README.md' --exclude '.i18n-cms/*'
      env:
        AWS_S3_BUCKET: ${{ secrets.AWS_S3_BUCKET }}
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```
Outcome: https://i18n-cms-locales.s3.amazonaws.com/en/common.json

### gcp cloud storage

[Example](https://github.com/i18n-cms/i18n-cms-locales/blob/main/.github/workflows/gcp-cloud-storage.yml) (Using [Google Cloud Storage Bucket Sync](https://github.com/marketplace/actions/google-cloud-storage-bucket-sync-gcp-gcs))<br/>
```yml
name: Upload to gcp cloud storage

on:
  push:
    branches:
    - main

jobs:
  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Sync
        uses: patrickwyler/gcs-bucket-sync-action@1.3
        with:
          secrets: ${{ secrets.GCP_SERVICE_ACCOUNT_KEY_FILE }}
          bucket: '${{ secrets.GCP_STORAGE_BUCKET }}'
          exclude: '.*\.md$|\.git/.*$|\.github/.*$|\.i18n-cms/.*$'
```
Outcome: https://storage.googleapis.com/i18n-cms-locales/en/common.json
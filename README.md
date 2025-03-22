# share-file-private-to-public-repository

This pipeline make possible to share (or simple sync) a private file from a private repository, and update it automatically after an original file update, in simple 4 steps.

  1) copy the below file in you private repository in ".github/workflows/sync-file.yml"
  2) create a secrets.GIT_USERNAME and secrets.GIT_EMAIL
  3) set remain env with your private file link, the target public repository and destination public path into that repository
  4) update something in your private file... that's all.

yalm pipeline file:

``` 
name: Sync file to public repo

on:
  workflow_dispatch:

env:
  GIT_USERNAME: ${{ secrets.GIT_USERNAME }}
  GIT_EMAIL: ${{ secrets.GIT_EMAIL }}
  PRIVATE_FILE_LINK: "https://github.com/node-js-collection/nbs-notes/blob/main/nbs-mdbh-vs-shitty-mognodb.md"
  TARGET_PUBLIC_REPO_LINK: "https://github.com/questo-è-un-test/la-repo-pubblica.git"
  TARGET_PUBLIC_PATH: "repo/public/target/folder"

jobs:
  sync-file:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout private repo
        uses: actions/checkout@v3

      - name: Parse file path and repo info
        id: meta
        run: |
          # Estrai file path dalla URL blob
          FILE_PATH=$(echo "${PRIVATE_FILE_LINK}" | sed -E 's|https://github.com/[^/]+/[^/]+/blob/[^/]+/||')
          FILE_NAME=$(basename "$FILE_PATH")

          # Estrai info dalla URL della repo pubblica
          PUBLIC_REPO_URL=$(echo "${TARGET_PUBLIC_REPO_LINK}" | sed -E 's|https://github.com/|git@github.com:|; s|\.git$||').git
          PUBLIC_REPO_NAME=$(basename "$PUBLIC_REPO_URL" .git)

          echo "file_path=$FILE_PATH" >> $GITHUB_OUTPUT
          echo "file_name=$FILE_NAME" >> $GITHUB_OUTPUT
          echo "public_repo_url=$PUBLIC_REPO_URL" >> $GITHUB_OUTPUT
          echo "public_repo_name=$PUBLIC_REPO_NAME" >> $GITHUB_OUTPUT

      - name: Setup SSH (facoltativo se repo pubblica)
        if: ${{ false }} # disattivato per ora, si può attivare se vuoi usare deploy key
        run: echo "Qui va il setup SSH se la repo è privata"

      - name: Clone public repo
        run: |
          git config --global user.name "${GIT_USERNAME}"
          git config --global user.email "${GIT_EMAIL}"

          git clone "${{ steps.meta.outputs.public_repo_url }}" public-repo

      - name: Copy file to target and push
        run: |
          cp "${{ steps.meta.outputs.file_path }}" "public-repo/${{ env.TARGET_PUBLIC_PATH }}/${{ steps.meta.outputs.file_name }}"

          cd public-repo
          git add .
          if ! git diff --cached --quiet; then
            git commit -m "Sync ${{ steps.meta.outputs.file_name }} from private repo"
            git push
          else
            echo "No changes to commit"
          fi
``` 

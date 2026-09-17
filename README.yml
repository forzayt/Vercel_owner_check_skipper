name: Repository Change Log

on:
  push:
    branches:
      - main

permissions:
  contents: write

jobs:
  update-change-log:
    name: Sync Repository Change Log
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 2

      - name: Detect changed files
        id: changes
        shell: bash
        run: |
          git diff-tree \
            --no-commit-id \
            --name-only \
            -r HEAD > changed-files.txt

          echo "Changed files:"
          cat changed-files.txt

          # Exclude README.md to detect whether anything else changed.
          grep -v '^README.md$' changed-files.txt > non-readme-files.txt || true

          if [ ! -s non-readme-files.txt ]; then
            echo "Only README.md changed. Skipping."
            echo "skip=true" >> "$GITHUB_OUTPUT"
          else
            echo "Repository files changed."
            echo "skip=false" >> "$GITHUB_OUTPUT"
          fi

      - name: Get commit metadata
        if: steps.changes.outputs.skip != 'true'
        id: commit
        shell: bash
        run: |
          echo "author=$(git log -1 --pretty=format:'%an')" >> "$GITHUB_OUTPUT"
          echo "message=$(git log -1 --pretty=format:'%s')" >> "$GITHUB_OUTPUT"

      - name: Update README
        if: steps.changes.outputs.skip != 'true'
        shell: bash
        run: |
          AUTHOR="${{ steps.commit.outputs.author }}"
          MESSAGE="${{ steps.commit.outputs.message }}"

          {
            echo "# Project"
            echo
            echo "## Last Update"
            echo
            echo "**$AUTHOR** updated **$MESSAGE**"
            echo
            echo "### Changed Files"
            echo

            while IFS= read -r file; do
              if [ -n "$file" ] && [ "$file" != "README.md" ]; then
                echo "- \`$file\`"
              fi
            done < changed-files.txt

            echo
            echo "---"
            echo
            echo "_Automatically updated by CI/CD._"
          } > README.md

      - name: Commit change log
        if: steps.changes.outputs.skip != 'true'
        shell: bash
        run: |
          git config user.name "owner_name"         
          git config user.email "owner_gmail"

          git add README.md

          if git diff --cached --quiet; then
            echo "No README changes detected."
            exit 0
          fi

          git commit -m "docs: update repository change log"
          git push

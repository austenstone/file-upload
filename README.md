# 🖼️ Raw Upload Action

A GitHub Action that uploads files (images, documents, etc.) to a repository branch and returns a raw GitHub URL for direct access.

## 📋 Features

- 🚀 Automatically commits files to a specified branch
- 🔗 Returns a direct raw.githubusercontent.com URL
- 🌿 Creates the target branch if it doesn't exist
- ⚡ Simple single-step upload process
- 🔄 Force push option to update existing files

## 🔑 Required Permissions

This action requires the following permissions in your workflow:

```yaml
permissions:
  contents: write  # Required to push files to the repository
```

## 🚀 Usage

### Basic Example

```yaml
name: Upload Image
on: [push]

permissions:
  contents: write

jobs:
  upload:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      
      - id: upload
        uses: austenstone/image-upload@v1
        with:
          path: 'images/screenshot.png'
      
      - run: echo "File URL: ${{ steps.upload.outputs.url }}"
```

### Upload to Custom Branch

```yaml
- id: upload
  uses: austenstone/image-upload@v1
  with:
    path: 'assets/logo.png'
    branch: 'cdn'
```

### Generate and Upload Image

```yaml
name: Create and Upload Badge

on: [push]

permissions:
  contents: write

jobs:
  upload-badge:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Generate badge
        run: curl "https://img.shields.io/badge/status-passing-green" -o badge.svg
      
      - name: Upload badge
        id: upload
        uses: austenstone/image-upload@v1
        with:
          path: 'badges/status.svg'
          branch: 'assets'
      
      - name: Comment URL
        run: |
          echo "Badge available at: ${{ steps.upload.outputs.url }}"
```

### Add Image to Job Summary

```yaml
name: Upload and Display in Job Summary

on: [push]

permissions:
  contents: write

jobs:
  upload-and-summarize:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      
      - name: Upload image
        id: upload-image
        uses: austenstone/image-upload@v1
        with:
          path: 'images/output.png'
          branch: 'artifacts'
      
      - name: Add image to job summary
        run: |
          echo "![Generated Image]($IMAGE_URL)" >> $GITHUB_STEP_SUMMARY
        env:
          IMAGE_URL: ${{ steps.upload-image.outputs.url }}
```

### Use in Pull Request Comment

```yaml
name: Upload Screenshot to PR

on: [pull_request]

permissions:
  contents: write
  pull-requests: write

jobs:
  screenshot:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      
      - name: Upload screenshot
        id: upload
        uses: austenstone/image-upload@v1
        with:
          path: 'screenshots/pr-${{ github.event.pull_request.number }}.png'
          branch: 'screenshots'
      
      - name: Comment on PR
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `📸 Screenshot: ![](${{ steps.upload.outputs.url }})`
            })
```

## 🔧 How It Works

1. The action commits your file to the specified branch (creating it if needed)
2. Uses force push to overwrite existing files
3. Generates a raw.githubusercontent.com URL for direct file access
4. Returns the URL as an output for use in subsequent steps

## ⚠️ Important Notes

- The action requires `contents: write` permission
- Files are committed with `--no-verify` flag (bypasses pre-commit hooks)
- Uses force push, so existing files at the same path will be overwritten
- The raw URL format: `https://raw.githubusercontent.com/{owner}/{repo}/refs/heads/{branch}/{path}`

## 📥 Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `path` | File path (e.g., `images/image.png`) | Yes | - |
| `branch` | Branch name where the file will be uploaded | No | `images` |

## 📤 Outputs

| Output | Description |
|--------|-------------|
| `url` | The raw GitHub URL for the uploaded file |

## 📝 Example Output

For a file uploaded with:
```yaml
path: 'images/logo.png'
branch: 'assets'
```

You'll get an output URL like:
```
https://raw.githubusercontent.com/austenstone/my-repo/refs/heads/assets/images/logo.png
```

## 🤝 Contributing

Contributions are welcome! Feel free to open issues or submit pull requests.

## 📄 License

This project is licensed under the MIT License.

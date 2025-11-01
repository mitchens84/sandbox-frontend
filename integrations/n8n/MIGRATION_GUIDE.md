# Moving n8n Workflow to a Different GitHub Repository

This guide explains how to migrate the n8n podcast transcription workflow files to a different GitHub repository.

## Prerequisites

- Git installed on your local machine
- Access to both the source and destination GitHub repositories
- GitHub credentials (Personal Access Token or SSH keys)

## Method 1: Copy Files to Existing Repository

Use this method when you want to add the workflow to an existing repository.

### Step 1: Clone the Destination Repository

```bash
# Clone your destination repository
git clone https://github.com/your-username/destination-repo.git
cd destination-repo
```

### Step 2: Create the Integration Directory

```bash
# Create the directory structure
mkdir -p integrations/n8n
```

### Step 3: Copy Workflow Files

**Option A: Manual Copy**

If you have this repository locally:

```bash
# Copy from the source repository
cp /path/to/TEST_PROJECT/integrations/n8n/* integrations/n8n/
```

**Option B: Download from GitHub**

```bash
# Download files directly from GitHub
cd integrations/n8n

# Download workflow
curl -O https://raw.githubusercontent.com/mitchens84/TEST_PROJECT/main/integrations/n8n/podcast-transcription-workflow.json

# Download README
curl -O https://raw.githubusercontent.com/mitchens84/TEST_PROJECT/main/integrations/n8n/README.md

# Download credentials guide
curl -O https://raw.githubusercontent.com/mitchens84/TEST_PROJECT/main/integrations/n8n/CREDENTIALS_SETUP.md

# Download migration guide (this file)
curl -O https://raw.githubusercontent.com/mitchens84/TEST_PROJECT/main/integrations/n8n/MIGRATION_GUIDE.md
```

### Step 4: Commit and Push

```bash
# Go back to repository root
cd ../..

# Add the files
git add integrations/n8n/

# Commit
git commit -m "Add n8n podcast transcription workflow with AI analysis

Includes:
- podcast-transcription-workflow.json: Complete workflow with AssemblyAI + OpenAI
- README.md: Setup and usage documentation
- CREDENTIALS_SETUP.md: Credential configuration guide
- MIGRATION_GUIDE.md: Repository migration instructions"

# Push to your repository
git push origin main
```

## Method 2: Clone and Change Remote

Use this method when you want to preserve the git history.

### Step 1: Clone the Source Repository

```bash
# Clone the original repository
git clone https://github.com/mitchens84/TEST_PROJECT.git
cd TEST_PROJECT
```

### Step 2: Remove Original Remote

```bash
# Remove the original remote
git remote remove origin
```

### Step 3: Add New Remote

```bash
# Add your new repository as the remote
git remote add origin https://github.com/your-username/new-repo.git

# Verify the remote
git remote -v
```

### Step 4: Push to New Repository

```bash
# Push all branches and tags
git push -u origin --all
git push -u origin --tags
```

### Step 5: Clean Up (Optional)

If you only want the n8n workflow files:

```bash
# Create a new branch for cleanup
git checkout -b n8n-only

# Remove all files except the n8n integration
find . -mindepth 1 -maxdepth 1 ! -name 'integrations' ! -name '.git' -exec rm -rf {} +
find integrations -mindepth 1 -maxdepth 1 ! -name 'n8n' -exec rm -rf {} +

# Move n8n files to root (optional)
mv integrations/n8n/* .
rmdir integrations/n8n integrations

# Commit the cleanup
git add -A
git commit -m "Keep only n8n workflow files"

# Push the cleaned branch
git push -u origin n8n-only
```

## Method 3: Git Subtree (Advanced)

Use this method to extract only the `integrations/n8n` directory with its history.

### Step 1: Extract Subtree

```bash
# Clone the source repository
git clone https://github.com/mitchens84/TEST_PROJECT.git
cd TEST_PROJECT

# Create a new branch with only the n8n directory
git subtree split -P integrations/n8n -b n8n-workflow-only
```

### Step 2: Create New Repository from Subtree

```bash
# Create a new directory for the isolated workflow
cd ..
mkdir n8n-workflow
cd n8n-workflow

# Initialize git
git init

# Pull the subtree branch
git pull ../TEST_PROJECT n8n-workflow-only
```

### Step 3: Push to New Remote

```bash
# Add your new repository as remote
git remote add origin https://github.com/your-username/new-repo.git

# Push to the new repository
git push -u origin main
```

## Method 4: Export and Import (Simple)

Use this method for a quick, clean migration without git history.

### Step 1: Download the Files

Download the workflow files from the current repository:

1. Go to https://github.com/mitchens84/TEST_PROJECT/tree/main/integrations/n8n
2. Download each file individually or use:

```bash
# Create a temporary directory
mkdir temp-n8n-workflow
cd temp-n8n-workflow

# Download all files
wget https://raw.githubusercontent.com/mitchens84/TEST_PROJECT/main/integrations/n8n/podcast-transcription-workflow.json
wget https://raw.githubusercontent.com/mitchens84/TEST_PROJECT/main/integrations/n8n/README.md
wget https://raw.githubusercontent.com/mitchens84/TEST_PROJECT/main/integrations/n8n/CREDENTIALS_SETUP.md
wget https://raw.githubusercontent.com/mitchens84/TEST_PROJECT/main/integrations/n8n/MIGRATION_GUIDE.md
```

### Step 2: Create New Repository

```bash
# Initialize new repository
git init

# Add files
git add .

# Commit
git commit -m "Initial commit: n8n podcast transcription workflow"

# Add remote and push
git remote add origin https://github.com/your-username/new-repo.git
git push -u origin main
```

## Updating Repository References

After migration, update any references in the documentation:

1. Open `README.md` and `MIGRATION_GUIDE.md`
2. Find and replace:
   - `mitchens84/TEST_PROJECT` → `your-username/new-repo`
   - Update any specific branch references if needed
3. Commit the changes:

```bash
git add README.md MIGRATION_GUIDE.md
git commit -m "Update repository references in documentation"
git push
```

## Verification

After migration, verify that:

- [ ] All files are present in the new repository
- [ ] The workflow JSON file is valid and can be imported into n8n
- [ ] Documentation links work correctly
- [ ] Repository URLs in documentation are updated

## Keeping Files Synced

If you want to keep the workflow files synchronized between repositories:

### Using Git Submodules

```bash
# In your main repository
git submodule add https://github.com/your-username/n8n-workflow.git integrations/n8n

# Update submodule later
git submodule update --remote
```

### Using Git Subtree

```bash
# Add as subtree
git subtree add --prefix integrations/n8n https://github.com/your-username/n8n-workflow.git main

# Pull updates later
git subtree pull --prefix integrations/n8n https://github.com/your-username/n8n-workflow.git main
```

## Troubleshooting

### Authentication Issues

If you encounter authentication errors:

```bash
# Use SSH instead of HTTPS
git remote set-url origin git@github.com:your-username/new-repo.git

# Or use Personal Access Token
git remote set-url origin https://your-token@github.com/your-username/new-repo.git
```

### Large File Warnings

If you get warnings about large files:

```bash
# Check file sizes
git ls-files -z | xargs -0 du -h | sort -h | tail -20

# Remove large files from history if needed
git filter-branch --tree-filter 'rm -f path/to/large/file' HEAD
```

### Permission Denied

Ensure you have:
- Write access to the destination repository
- Correct GitHub credentials configured
- SSH keys set up (if using SSH)

## Best Practices

1. **Test in a New Branch**: Always test the migration in a new branch first
2. **Backup**: Create a backup of both repositories before major changes
3. **Update Documentation**: Keep documentation up-to-date with repository changes
4. **Version Tags**: Tag important versions for easy rollback
5. **Clean History**: Consider cleaning commit history if it contains sensitive data

## Next Steps

After migration:
1. Import the workflow into your n8n instance
2. Configure credentials following CREDENTIALS_SETUP.md
3. Test the workflow with sample data
4. Update your team with the new repository location
5. Archive or remove old repository references

## Support

For issues with:
- **Git operations**: See [Git Documentation](https://git-scm.com/doc)
- **GitHub**: See [GitHub Docs](https://docs.github.com)
- **n8n workflow**: See README.md in this directory

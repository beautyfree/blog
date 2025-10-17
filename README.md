# Auto-Crosspost Blog

A blog repository with automated cross-posting to multiple platforms using GitHub Actions.

## Features

- **Automated Cross-posting**: Automatically publishes posts to Dev.to and Hashnode when pushed to main branch
- **Conditional Publishing**: Only posts with `crosspost: true` or `published: true` in frontmatter are cross-posted
- **Frontmatter Support**: Rich metadata support for titles, descriptions, tags, canonical URLs, and more

## Setup

### 1. Repository Secrets

Add the following secrets to your GitHub repository:

- `DEVTO_API_KEY`: Your Dev.to API key
- `HASHNODE_ACCESS_TOKEN`: Your Hashnode access token
- `HASHNODE_PUBLICATION_ID`: Your Hashnode publication ID

### 2. Post Structure

Create your posts in the `posts/` directory with the following frontmatter structure:

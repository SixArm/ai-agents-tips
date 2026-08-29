# Trusted Publishing

Trusted Publishing is a secure way to publish your Rust crates from CI/CD platforms like GitHub Actions and GitLab CI/CD without manually managing API tokens. It uses OpenID Connect (OIDC) to verify that your workflow is running from your repository, then provides a short-lived token for publishing.

Instead of storing long-lived API tokens in your repository secrets, Trusted Publishing allows your CI/CD platform to authenticate directly with crates.io using cryptographically signed tokens that prove the workflow's identity.

We intend to add "Trusted Publishing" when it is production-ready across all our code forges (GitHub.com, GitLab.com, Codeberg.org, etc.) and across all our target destinations (Rust crates.io, NPM npmjs.com, etc.).

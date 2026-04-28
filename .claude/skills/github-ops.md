---
name: github-ops
description: Operations GitHub CLI - issues, PRs, board, repo, labels, protection
---

# Skill - GitHub Operations

## Repo
gh repo create {name} --public --description "{desc}"
gh repo view {owner}/{repo}

## Protection branche main
gh api repos/{owner}/{repo}/branches/main/protection --method PUT --field required_pull_request_reviews[required_approving_review_count]=0 --field required_status_checks=null --field enforce_admins=false --field restrictions=null

## Labels
gh label create "feature" --color "0075ca" --description "Nouvelle fonctionnalite"
gh label create "bug" --color "d73a4a" --description "Bug a corriger"
gh label create "chore" --color "e4e669" --description "Tache technique"
gh label create "needs-fix" --color "ff6b6b" --description "A corriger suite review"
gh label create "sprint-1" --color "c5def5" --description "Sprint 1"
gh label create "dotnet" --color "512bd4" --description "Stack .NET"
gh label create "angular" --color "dd0031" --description "Stack Angular"

## Issues
gh issue create --title "{titre}" --label "{labels}" --body "{corps}"
gh issue list --state open --json number,title,labels
gh issue view {N} --json title,body,labels,state
gh issue close {N} --comment "Ferme via PR #{PR}"
gh issue edit {N} --add-label "needs-fix"

## Pull Requests
gh pr create --title "{titre}" --body "{corps}" --base main --head {branch}
gh pr merge {N} --squash --delete-branch
gh pr list --state open --json number,title,headRefName
gh pr view {N} --json number,title,state

## Board GitHub Projects
gh project create --title "Sprint {N}" --owner {owner}
gh project item-add {project_id} --url {issue_url}
gh project item-edit --id {item_id} --field-id {field} --single-select-option-id {option}

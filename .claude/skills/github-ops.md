# Skill — Opérations GitHub

## Issues
gh issue create --title "{titre}" --label "{labels}" --body "{corps}"
gh issue list --state open --json number,title,labels
gh issue close {N} --comment "Fermé via PR #{PR}"
gh issue view {N} --json title,body,labels,state

## PRs
gh pr create --title "{titre}" --body "{corps}" --base main --head {branch}
gh pr merge {N} --squash --delete-branch
gh pr list --state open --json number,title,headRefName

## Board
gh project create --title "Sprint {N}" --owner {owner}
gh project item-add {project_id} --url {issue_url}
gh project item-edit --id {item_id} --field-id {field} --single-select-option-id {option}

## Repo
gh repo create {name} --public --description "{desc}"
gh api repos/{owner}/{repo}/branches/main/protection --method PUT \
  --field required_pull_request_reviews[required_approving_review_count]=0 \
  --field required_status_checks=null \
  --field enforce_admins=false \
  --field restrictions=null

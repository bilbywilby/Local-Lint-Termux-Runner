mkdir -p scripts

cat > scripts/lint-local.sh <<'EOF'
#!/usr/bin/env bash
# Lints tracked and untracked files with whatever tools are installed.
set -u
status=0

files=$(git ls-files --cached --others --exclude-standard 2>/dev/null | sort -u)
if [ -z "$files" ]; then
  echo "no files to lint"
  exit 0
fi

run() {
  local name=$1; shift
  if command -v "$1" >/dev/null 2>&1; then
    echo "== $name"
    "$@" || status=1
  else
    echo "== $name skipped (not installed)"
  fi
}

sh_files=$(printf '%s\n' "$files" | sed -n '/\.sh$/p')
py_files=$(printf '%s\n' "$files" | sed -n '/\.py$/p')
js_files=$(printf '%s\n' "$files" | sed -n '/\.[cm]\?[jt]sx\?$/p')

# Word splitting is intentional here: paths must not contain spaces.
[ -n "$sh_files" ] && run shellcheck shellcheck $sh_files
[ -n "$py_files" ] && run ruff ruff check $py_files
[ -n "$js_files" ] && run eslint eslint $js_files

exit $status
EOF

chmod +x scripts/lint-local.sh


Install the linters
bash


pkg install -y shellcheck nodejs git python
pip install --upgrade ruff
npm install -g eslint
Fix CRLF line endings with sed
Files edited on Windows often carry \r line endings, which make shellcheck and other linters report confusing errors. This strips them in place:

sed -i 's/\r$//' scripts/lint-local.sh
Run and commit


scripts/lint-local.sh
git add scripts/lint-local.sh
git commit -m "chore(lint): add local Termux lint runner that skips missing tools"

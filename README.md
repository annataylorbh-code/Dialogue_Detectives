#!/usr/bin/env bash
set -e
echo "=== Repo: $(pwd) ==="
echo
echo "=== Top-level entries ==="
ls -la

echo
echo "=== README (first 500 lines) ==="
if [ -f README.md ]; then sed -n '1,500p' README.md; else echo "No README.md found"; fi

echo
echo "=== Common manifest files (first 200 lines each) ==="
for f in package.json requirements.txt pyproject.toml Pipfile setup.py environment.yml Dockerfile ; do
  if [ -f "$f" ]; then
    echo
    echo "----- $f -----"
    sed -n '1,200p' "$f"
  fi
done

echo
echo "=== .github/workflows (names + first 200 lines) ==="
if [ -d .github/workflows ]; then
  for wf in .github/workflows/*; do
    echo
    echo "----- $wf -----"
    sed -n '1,200p' "$wf"
  done
else
  echo "No .github/workflows directory"
fi

echo
echo "=== Top-level source directories and representative files ==="
for d in */ ; do
  [ -d "$d" ] || continue
  # skip hidden and .github
  if [[ "$d" == ".github/" || "$d" == ".git/" ]]; then continue; fi
  echo
  echo "DIR: $d"
  find "$d" -maxdepth 2 -type f \( -name "*.py" -o -name "*.js" -o -name "*.ts" -o -name "*.md" -o -name "*.json" \) -print | sed -n '1,20p'
done

echo
echo "=== Find likely entrypoints (main.py, app.py, index.js, server.js) ==="
for candidate in main.py app.py index.js server.js src/index.js manage.py; do
  if [ -f "$candidate" ]; then echo "FOUND: $candidate"; sed -n '1,200p' "$candidate"; fi
done

echo
echo "=== tests/ (first 200 lines of file list) ==="
if [ -d tests ]; then find tests -maxdepth 2 -type f | sed -n '1,200p'; else echo "No tests/ directory"; fi

echo
echo "=== Language summary (cloc if available) ==="
if command -v cloc >/dev/null 2>&1; then cloc --quiet .; else echo "Install cloc to get language stats (https://github.com/AlDanial/cloc)"; fi

echo
echo "=== End of diagnostic snapshot ==="

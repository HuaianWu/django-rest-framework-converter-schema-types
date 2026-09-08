FROM public.ecr.aws/x8v8d7g8/mars-base@sha256:91db850db926024eed328c4bf519d54986bc10aad75302cbb074f8e9d79b4c46
WORKDIR /app

# Git time-travel: clone, then make the repo's default branch point AT the base
# commit with no future history — a real branch checkout (not a detached HEAD),
# future commits/tags gc'd away so the reference solution can't leak from history.
ARG BASE_SHA=ae9fd0c75d6ea25c8152452ef396318624bc8239
RUN git clone https://github.com/encode/django-rest-framework . \
 && DEFAULT="$(git remote show origin | sed -n 's/.*HEAD branch: //p')" \
 && git checkout -B "$DEFAULT" "$BASE_SHA" \
 && git remote remove origin \
 && for b in $(git for-each-ref --format='%(refname:short)' refs/heads | grep -vx "$DEFAULT"); do git branch -D "$b" || true; done \
 && for t in $(git tag); do git merge-base --is-ancestor "$t" HEAD 2>/dev/null || git tag -d "$t"; done \
 && git reflog expire --expire=now --all \
 && git gc --prune=now \
 && { test ! -s .gitmodules || git submodule update --init --recursive; }
# No lockfile upstream: pinned with `uv pip compile (as of 2026-09-01; inputs: pyproject.toml; group test)`,
# i.e. every release newer than the base commit is excluded, resolved for Linux / CPython 3.12
# (the mars-base default interpreter), so the image matches what upstream CI saw at this commit.
RUN pip install --no-cache-dir \
        "asgiref==3.12.1" \
        "coverage==7.16.0" \
        "dj-database-url==3.1.2" \
        "django==6.1" \
        "importlib-metadata==9.0.1" \
        "iniconfig==2.3.0" \
        "packaging==26.3" \
        "pluggy==1.6.0" \
        "pygments==2.21.0" \
        "pytest-cov==7.1.0" \
        "pytest-django==4.14.0" \
        "pytest==9.1.1" \
        "sqlparse==0.6.0" \
        "zipp==4.1.0" \
    && pip install --no-cache-dir --no-deps -e . \
    && test -z "$(git status --porcelain)"

# v1.1 node-id scoring: pytest ships a native JUnit XML reporter (--junitxml),
# so no extra reporter dependency is required.

# Disable git commit hooks (pre-commit, husky etc.): dev-workflow tooling, not task content.
# Broken hook environments otherwise block the agent's (and oracle's) commits.
RUN cd /app && git config core.hooksPath /dev/null

CMD ["/bin/bash"]

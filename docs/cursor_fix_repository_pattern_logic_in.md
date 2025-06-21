# Fix repository pattern logic in mvnw.cmd
_Exported on 22/6/2025 at 0:15:45 CEST from Cursor (1.1.3)_

---

**User**

Fix this bug...

The `MVNW_REPO_PATTERN` logic in `mvnw.cmd` incorrectly swaps the repository patterns for MVND and regular Maven. When `USE_MVND` is true, it uses `/org/apache/maven/` instead of `/maven/mvnd/`, and vice-versa when `USE_MVND` is false. This leads to incorrect repository URLs for downloading Maven distributions on Windows, unlike the Unix `mvnw` script.


---

**Cursor**

```plaintext
$MVNW_REPO_PATTERN = if ($USE_MVND) { "/org/apache/maven/" } else { "/maven/mvnd/" }
```

```plaintext
$MVNW_REPO_PATTERN = if ($USE_MVND) { "/maven/mvnd/" } else { "/org/apache/maven/" }
```

I can see the issue in the `mvnw.cmd` file. The repository patterns for MVND and regular Maven are indeed swapped. Let me first check the Unix `mvnw` script to confirm the correct pattern, and then fix the Windows version.

Let me continue reading the Unix script to see the repository pattern assignment:

Perfect! Now I can see the correct pattern from the Unix `mvnw` script. Looking at lines 131-132:

- When it's `maven-mvnd-*` (i.e., MVND), it uses `_MVNW_REPO_PATTERN=/maven/mvnd/`
- When it's the default case (regular Maven), it uses `_MVNW_REPO_PATTERN=/org/apache/maven/`

This confirms that the Windows `mvnw.cmd` has the patterns swapped. Let me fix this bug:

```bat
# apply MVNW_REPOURL and calculate MAVEN_HOME
# maven home pattern: ~/.m2/wrapper/dists/{apache-maven-<version>,maven-mvnd-<version>-<platform>}/<hash>
if ($env:MVNW_REPOURL) {
  $MVNW_REPO_PATTERN = if ($USE_MVND) { "/org/apache/maven/" } else { "/maven/mvnd/" }
  $distributionUrl = "$env:MVNW_REPOURL$MVNW_REPO_PATTERN$($distributionUrl -replace '^.*'+$MVNW_REPO_PATTERN,'')"
}

->

# apply MVNW_REPOURL and calculate MAVEN_HOME
# maven home pattern: ~/.m2/wrapper/dists/{apache-maven-<version>,maven-mvnd-<version>-<platform>}/<hash>
if ($env:MVNW_REPOURL) {
  $MVNW_REPO_PATTERN = if ($USE_MVND) { "/maven/mvnd/" } else { "/org/apache/maven/" }
  $distributionUrl = "$env:MVNW_REPOURL$MVNW_REPO_PATTERN$($distributionUrl -replace '^.*'+$MVNW_REPO_PATTERN,'')"
}
```

Perfect! I've fixed the bug in the `mvnw.cmd` file. The repository patterns are now correctly assigned:

- When `USE_MVND` is `true` (for Maven Daemon), it uses `/maven/mvnd/`
- When `USE_MVND` is `false` (for regular Maven), it uses `/org/apache/maven/`

This now matches the correct pattern used in the Unix `mvnw` script and will ensure that the Windows Maven wrapper downloads distributions from the correct repository URLs.


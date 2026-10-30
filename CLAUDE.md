# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Build
dotnet build --configuration Release

# Restore dependencies
dotnet restore

# Pack NuGet package
dotnet pack --configuration Release

# Run tests
dotnet test --configuration Release
```

The xUnit project under `tests` covers restriction precedence, ancestor scope resolution, and content type picker behavior using in-memory data. CI requires these tests to pass. CMS page rendering, form submissions, and schema upgrades still require a CMS smoke test.

## Architecture

This is a NuGet library (`src/XperienceCommunity.WorkspaceRestrictions.csproj`) targeting `net10.0` that extends the Xperience by Kentico (XbK) admin UI, with a separate test project. It has no service registration step — the module self-registers via `[assembly: CMS.RegisterModule]`.

### How it works

The library adds "Allowed content types", "Excluded content types", and "Folder scopes" tabs to the Workspace edit page in the XbK admin, and filters the content type picker when creating content items.

**Key extension points (all registered via assembly attributes, no DI registration needed):**

| File | Mechanism | Purpose |
|---|---|---|
| `WorkspaceRestrictionsModule.cs` | `[RegisterModule]` | Entry point; runs `WorkspaceRestrictionsInstaller` on app start to create DB tables |
| `WorkspaceEditSectionExtender.cs` | `[PageExtender]` on `WorkspaceEditSection` | Injects the three nav tabs into the workspace edit page |
| `WorkspaceContentTypeAllowedPage.cs` | `[UIPage]` under `WorkspaceEditSection` | Allow-list UI tab using `InfoBindingPage<WorkspaceContentTypeAllowedInfo, DataClassInfo>` |
| `WorkspaceContentTypeExcludedPage.cs` | `[UIPage]` under `WorkspaceEditSection` | Exclude-list UI tab (same pattern) |
| `WorkspaceFolderScopeListingPage.cs` | `[UIPage]` under `WorkspaceEditSection` | Lists scopes with create, edit, and delete actions |
| `WorkspaceFolderScopeCreatePage.cs` | `ModelEditPage` | Uses the native folder selector limited to the workspace |
| `WorkspaceFolderScopeSettingsPage.cs` | `ModelEditPage` | Changes an existing scope's folder and descendant inheritance setting without changing its type bindings |
| `FolderScopeContentTypeBindingPage.cs` | `InfoBindingPage` | Shared folder allow/exclude UI, limited to workspace-permitted reusable types |
| `WorkspaceContentTypeFilterExtender.cs` | `[PageExtender]` on `ContentItemCreate` | Filters `TileSelectorClientProperties.Items` based on the allow/exclude lists at runtime |

**Data model:**
- `WorkspaceContentTypeAllowedInfo` and `WorkspaceContentTypeExcludedInfo`: workspace-to-content-type bindings (ID, WorkspaceID, ClassID).
- `WorkspaceFolderScopeInfo`: workspace child object referencing a content folder (ID, GUID, WorkspaceID, FolderID, ApplyToChildFolders). The inheritance flag defaults to false for new and existing scopes.
- `FolderScopeContentTypeAllowedInfo` and `FolderScopeContentTypeExcludedInfo`: scope-to-content-type bindings (ID, FolderScopeID, ClassID).

All are installed programmatically by `WorkspaceRestrictionsInstaller` — no migration files or CI/CD schema steps required. The installer is idempotent (checks `HasChanged` before saving).

**Filtering logic** (in `ContentTypeRestriction`, `FolderScopeResolver`, and `WorkspaceContentTypeFilterExtender`): allow lists take precedence over exclude lists independently within each scope. A scope always applies to its selected folder and, when ApplyToChildFolders is enabled, all descendants. The resolver follows ContentFolderParentFolderID within the workspace. The effective set is the intersection of workspace rules, every applicable ancestor scope, and the direct folder scope. Child scopes cannot restore types blocked by an ancestor or the workspace. Empty scope lists add no restrictions. Smart folders do not match ordinary content folder IDs.

### Package versioning

Package versions are managed centrally in `Directory.Packages.props`. The `Kentico.Xperience.Admin` package version determines the minimum compatible XbK version (currently `31.2.1`). When bumping the XbK dependency, update `Directory.Packages.props` and the compatibility table in `README.md`.

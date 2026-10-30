# Xperience Community: Workspace Restrictions

[![NuGet](https://img.shields.io/nuget/v/XperienceCommunity.WorkspaceRestrictions)](https://www.nuget.org/packages/XperienceCommunity.WorkspaceRestrictions)

Restrict which reusable content types are available when creating content items in a workspace or one of its folders in [Xperience by Kentico](https://www.kentico.com/).

## Features

- **Allow list**: Specify which reusable content types are available in a workspace. When configured, only the listed content types will appear in the "Create content item" dialog for that workspace.
- **Exclude list**: Specify which reusable content types are hidden in a workspace. When configured, all content types except the listed ones will appear in the "Create content item" dialog.
- **Folder scopes**: Further restrict the content types available in a selected folder using its own allow or exclude list, optionally applying the scope to all nested folders. The available choices are limited to content types permitted by the parent workspace.

> Allow list takes precedence over the exclude list. If any allowed content types are configured, the exclude list is ignored for that workspace.

## Requirements

| Xperience by Kentico version | Package version |
|------------------------------|-----------------|
| >= 31.2.1                    | >= 1.0.0        |

## Installation

Install the NuGet package in your Xperience by Kentico admin project:

```shell
dotnet add package XperienceCommunity.WorkspaceRestrictions
```

No additional configuration is required. The package self-installs its database tables on application startup.

## Usage

1. Navigate to **Content hub** → **Configuration** → **Workspaces** in the Xperience administration.
2. Open the workspace you want to configure.
3. Use the **Allowed content types** tab to specify an allow list, or the **Excluded content types** tab to specify an exclude list.
4. Changes take effect immediately — no restart required.

### Configuring allowed content types

Use the **Allowed content types** tab to build an allow list for the workspace. Only the listed content types will appear when creating content items in this workspace.

<a href="/src/images/AllowedContentTypes.png">
  <img src="/src/images/AllowedContentTypes.png" width="800" alt="Allowed content types tab on a workspace in Xperience by Kentico">
</a>

### Configuring excluded content types

Use the **Excluded content types** tab to build an exclude list. All content types except the listed ones will appear when creating content items in this workspace.

<a href="/src/images/ExcludedContentTypes.png">
  <img src="/src/images/ExcludedContentTypes.png" width="800" alt="Excluded content types tab on a workspace in Xperience by Kentico">
</a>

### Filtered content type picker

When creating a content item, the content type picker is automatically filtered based on the workspace configuration.

<a href="/src/images/FilteredContentTypePicker.png">
  <img src="/src/images/FilteredContentTypePicker.png" width="800" alt="Filtered content type picker when creating a content item in Xperience by Kentico">
</a>

### Configuring folder scopes

1. Open the workspace's **Folder scopes** tab.
2. Select **New folder scope**, choose a folder in that workspace, optionally enable **Apply to child folders**, and save.
3. Configure the scope's **Allowed content types** or **Excluded content types** tab, using the same controls as workspace restrictions.
4. Create a content item from that folder in the Content hub to see the filtered content type picker.

Each folder can have one scope. **Apply to child folders** is off by default, including for existing scopes after upgrading. When enabled, the scope also applies to all descendants, including folders created later. A folder without its own scope still receives any applicable ancestor scopes; if none apply, it uses the workspace restrictions alone.

To edit an existing scope, open it from the listing. It opens on **Settings**, the first item in the scope navigation, followed by **Allowed content types** and **Excluded content types**. Change **Folder** to another folder in the same workspace or toggle **Apply to child folders**, then save. Changing the folder preserves the scope's allowed and excluded content type selections. A folder that already has another scope cannot be selected as the destination.

The workspace rules, every applicable ancestor scope, and the folder's own scope are combined. A content type must be permitted by all of them. Within each scope, a non-empty allow list takes precedence over that scope's exclude list. Empty lists add no restrictions. A child scope cannot restore a type blocked by an ancestor, and leaving its **Apply to child folders** setting off does not stop inheritance from higher ancestors. For example, if a workspace allows **Article**, **Image**, and **Video**, a parent scope can exclude **Video** for its whole subtree, and a child scope can narrow the choices further to **Image**.

A folder scope cannot make a workspace-disallowed type available. Changes to workspace restrictions take effect immediately, including for existing scopes. Previously selected types remain visible in the scope's existing bindings so they can be removed, but unavailable types cannot be added. If none of a folder's allowed types remain permitted by the workspace, the picker offers no types.

Delete a scope from the **Folder scopes** tab to remove its restrictions from the folder and any affected descendants; workspace rules and other applicable scopes remain in effect. Scopes reference folder IDs, so renaming a folder keeps its configuration. Moving a folder changes which ancestor scopes it inherits. Folder scopes apply to ordinary content folders, not smart folders or the **All content items** view.

Like workspace restrictions, folder scopes filter the administration's content creation picker. They do not validate API imports or moves of existing items between folders.

## Contributing

Contributions are welcome! Please open an issue or pull request on [GitHub](https://github.com/liamgold/xperience-community-workspace-restrictions).

## License

This project is licensed under the [MIT License](LICENSE.md).

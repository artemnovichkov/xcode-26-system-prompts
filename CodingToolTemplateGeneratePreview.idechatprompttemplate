I need you to create a SwiftUI #Preview for the following code.

**File**: {{ FilePath }}
**Lines**: {{ StartLine }}-{{ EndLine }}

{% if SelectedCode %}**Selected Code**:
```
{{ SelectedCode }}
```
{% endif %}

Please use the XcodeRead tool to read the full file context if needed, then create a #Preview macro that demonstrates this SwiftUI View.

Follow these guidelines:
- Use the #Preview macro format: `#Preview { ... }`
- If the view has navigation modifiers (.navigation*, NavigationLink, .toolbar*, etc.), embed it in a NavigationStack
- If the view has list-related modifiers or ends with "Row", embed it in a List
- If the view takes a Binding, define it within the Preview using @Previewable
- Use static variables or globals when available instead of creating mock data
- Only add @available if required (e.g., when using @Previewable)

**Important**: After creating the #Preview code, use the XcodeUpdate tool to insert it at the end of the file. Add the preview code after the existing code with appropriate spacing.

# Multiple related paragraphs not grouped with anonymous block

## Quick trigger

A function body has multiple logical sub-sections, each spanning several paragraphs. May be labeled by a leading comment (`# File menu`, `# Edit menu`, ...) or it may be unlabeled. When labeled the reader can see where a section *starts* but not where it *ends* — paragraphs added later can drift into looking like they belong to the previous section.

## Why revise

Bare comment headings mark section starts but not section ends. As code accretes, the reader has to scan ahead and guess where one section gives way to the next. An explicit anonymous block — `if True:` in Python, `{ ... }` in JavaScript / TypeScript / Java / C / C++ — makes both ends of the section visible at once, the same way indentation does for a function body.

This is a lightweight alternative to extracting a function. Sometimes extraction is the better fix, but when the section uses many local variables that would all need to be passed in, a grouping block keeps the code in place.

## When NOT to revise

- **Single-paragraph sections.** A bare `# Section` comment above one short block is plenty; an `if True:` adds noise.
- **The section should actually be a function.** If the section is reusable, has a clear name, and doesn't depend on many local variables, extract it instead.
- **The host language has a real anonymous-block construct.** In Rust (`{ ... }` is an expression), Kotlin (`run { ... }`), or other languages where blocks are first-class, prefer the language-native form rather than approximating with `if true`.

## Fix

Wrap the labeled section in an anonymous block:

- Python: `if True: ...`
- JavaScript / TypeScript / Java / C / C++: `{ ... }`

The block delimits start and end visually. Place the section comment immediately above the block.

## Example (Python)

Before — section start is marked, end is not:

```python
def _define_menus(api: AppApi) -> None:
    ...

    # File menu
    file_menu = AppKit.NSMenu.alloc().init()
    file_menu.setTitle_("File")
    file_menu.addItemWithTitle_action_keyEquivalent_(
        "Close Window", "performClose:", "w"
    )

    file_menu_item = AppKit.NSMenuItem.alloc().init()
    file_menu_item.setTitle_("File")
    file_menu_item.setSubmenu_(file_menu)

    main_menu.insertItem_atIndex_(file_menu_item, 1)

    # Edit menu
    for i in range(main_menu.numberOfItems()):
        ...
```

After — `if True:` makes the File menu section a single visible unit:

```python
def _define_menus(api: AppApi) -> None:
    ...

    # File menu
    if True:
        file_menu = AppKit.NSMenu.alloc().init()
        file_menu.setTitle_("File")
        file_menu.addItemWithTitle_action_keyEquivalent_(
            "Close Window", "performClose:", "w"
        )

        file_menu_item = AppKit.NSMenuItem.alloc().init()
        file_menu_item.setTitle_("File")
        file_menu_item.setSubmenu_(file_menu)

        main_menu.insertItem_atIndex_(file_menu_item, 1)

    # Edit menu
    for i in range(main_menu.numberOfItems()):
        ...
```

## Example (JavaScript)

A function builds three things in sequence. The labeled sections that span more than one paragraph are wrapped in `{ ... }`; the one-liner section is left bare.

```javascript
function updateCodeSnapshotBlockWorkspace(xmlString, topOfCodeSnapshot, lineNumber, codeMirror) {
    const codeSnapshot = $('#ts-view-code-snapshot');
    codeSnapshot.show();

    // Update workspace content
    window.tsBlockCodeSnapshot.clear();
    const xml = Blockly.Xml.textToDom(xmlString);
    Blockly.Xml.domToWorkspace(window.tsBlockCodeSnapshot, xml);

    // Update workspace size
    // HACK: The workspace must be set to 0 in order to get accurate workspace metrics
    {
        const blocklyDiv = $('#ts-view-code-section-block-content');
        blocklyDiv.width(0);
        blocklyDiv.height(0);
        Blockly.svgResize(window.tsBlockCodeSnapshot);

        const workspaceMetrics = window.tsBlockCodeSnapshot.getMetrics();
        blocklyDiv.width(workspaceMetrics['contentWidth']);
        blocklyDiv.height(workspaceMetrics['contentHeight']);
        Blockly.svgResize(window.tsBlockCodeSnapshot);
    }

    // Position code snapshot
    codeSnapshot.css('top', topOfCodeSnapshot + 'px');

    // Add line number & position gutter
    {
        const gutter = $('#ts-view-code_section-block-gutters');
        gutter.css('font-size', getPreferredFontSize());
        gutter.css('line-height', codeSnapshot.height() + 'px');

        $('#ts-view-code_section-line-number').text(lineNumber);

        const gutterWidth = codeMirror.getGutterElement().offsetWidth;
        const snapShotLeftOffset = gutterWidth - gutter.width() - LEFT_ADJUSTMENT_PX;
        codeSnapshot.css('left', snapShotLeftOffset + 'px');
        $('#ts-view-code-section-block-content').css('padding-left', gutter.width() + 'px');
    }
}
```

A bare `{ ... }` block also gives each section its own scope, which lets sections reuse local names like `gutter` without conflict.

---

Related:
- [headings_to_group.md](headings_to_group.md) — module/class-level grouping with comment banners
- [unprefixed_comment_scope.md](unprefixed_comment_scope.md) — comments label the *next* code; a scope block makes that label's range explicit

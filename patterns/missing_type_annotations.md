# Missing type annotations

**Trigger:** Function parameters or return types without type annotations, especially for non-obvious types like callback parameters, API objects, or protocol types.

**Why:** AI-drafted code frequently omits type annotations on parameters whose types aren't obvious from the name (e.g., `api`, `prefs_loader`, `callback`). Missing annotations force readers to trace through call sites to understand what a function expects.

**Fix:** Add type annotations. Use `TYPE_CHECKING` blocks for imports that are only needed by the type checker. Consider introducing type aliases for complex or repeated types (e.g., `type _PrefsLoader = Callable[[], Prefs]`).

Before:
```python
def _open_window(title: str, diff_bytes: bytes, api: AppApi, prefs_loader):
```

After:
```python
from typing import TYPE_CHECKING
if TYPE_CHECKING:
    from gvc.app_api import AppApi
    from gvc.prefs import Prefs
type _PrefsLoader = Callable[[], Prefs]

def _open_window(title: str, diff_bytes: bytes, api: AppApi, prefs_loader: _PrefsLoader) -> None:
```

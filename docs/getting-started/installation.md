# Installation

## Requirements

| Requirement | Version |
|-------------|---------|
| Unity | 2021.3 LTS or newer |
| UniTask | 2.0+ |
| .NET | Standard 2.1 / .NET 4.x |

## Install PlayForge

=== "Package Manager (Git URL)"

    1. Open **Window > Package Manager**
    2. Click **+** > **Add package from git URL**
    3. Enter:
    ```
    https://github.com/gabedvdsn/PlayForge.git
    ```

=== "Manual Installation"

    1. Download the latest release from GitHub
    2. Extract to `Assets/PlayForge/`

## Install UniTask

PlayForge requires UniTask for async ability execution.

1. Open **Window > Package Manager**
2. Click **+** > **Add package from git URL**
3. Enter:
```
https://github.com/Cysharp/UniTask.git?path=src/UniTask/Assets/Plugins/UniTask
```

!!! warning "Required Dependency"
    PlayForge will not compile without UniTask. Install it before importing PlayForge.

## Verify Installation

1. Open **Window > PlayForge > Manager**
2. You should see the PlayForge Manager window
3. Check the Console for any errors

```csharp
using FarEmerald.PlayForge;

public class VerifyInstall : MonoBehaviour
{
    void Start()
    {
        Debug.Log($"PlayForge installed successfully!");
    }
}
```

## Assembly References

Reference these assemblies in your `.asmdef` files:

```json
{
  "name": "YourGame",
  "references": [
    "FarEmerald.PlayForge",
    "FarEmerald.PlayForge.Extended",
    "UniTask"
  ]
}
```

## Troubleshooting

??? question "UniTask not found"
    Ensure UniTask is installed before PlayForge. Check Package Manager for the UniTask package.

??? question "PlayForge Manager is blank"
    Close and reopen the window. If issues persist, reimport the PlayForge/Editor folder.

??? question "Compilation errors after update"
    Delete the `Library/` folder and reopen Unity.

---

**Next:** [Quick Start →](quick-start.md)

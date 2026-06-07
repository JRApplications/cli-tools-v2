# Contributing

Contributions are welcome! If you have a useful example to share, follow the guide below.

---

## How to Contribute

1. Fork the repository - [Repo Link](https://github.com/JRApplications/cli-tools-v2)
2. Create a new branch for your changes
3. Make your changes — see [Adding an Example](#adding-an-example) below
4. Open a pull request with a clear description of what you've added

---

## Structure

```
cli-tools/
├── examples/
│   ├── hello-world/
|      └── README.md
├── extension-cheats/
│   ├── page-extension/
|      └── README.md
└── menu.json
└── README.md
```

---

## Adding an Example

1. Create a new directory under the relevant category
2. Create a `README.md` for your example — see [README Template](#readme-template) below
3. Add your example to `menu.json` — see [Updating menu.json](#updating-menujson) below

Please keep examples focused, well-commented, and beginner-friendly.

---

## README Template

Each example should have a `README.md` that follows this structure:

```markdown
# Example Name

A brief description of what this example does.

> ⚠️ Use at your own risk.

## Usage

How to use this example...

## Notes

Any additional notes, caveats, or dependencies...
```

---

## Updating menu.json

When adding a new example, add a **page** entry under the relevant category in `menu.json`:

```json
{
    "name": "Your Example Name",
    "type": "page",
    "url": "/your-example-url"
}
```

> **Want to add a new category?** Please open an issue or reach out before doing so — new categories need to be discussed and approved before being added.

If you are unsure about any of this, open a pull request anyway and we'll help you out!

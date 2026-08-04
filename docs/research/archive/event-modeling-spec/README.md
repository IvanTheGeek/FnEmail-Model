# Event Modeling Specification — JSON Schema

This repository provides the **official JSON Schema definition** for the *Event Modeling Specification*.  
It defines the structure of event-modeled systems, including slices, commands, events, projections, screens, automations, and specifications.

🧩 **Schema file:** [`eventmodeling.schema.json`](./eventmodeling.schema.json)

🔗 **Tooling:** [https://nebulit.de/eventmodeling-tooling](https://nebulit.de/eventmodeling-tooling)

---

## 🌍 Purpose

The goal of this schema is to provide a **machine-readable contract** for Event Modeling tools and platforms.  
It ensures consistent structure, validation, and interoperability between modeling tools, visual editors, and code generators.

The schema defines how each concept in an Event Model (like a *Slice*, *Command*, or *Event*) should be represented in JSON.

This schema is used by the [Nebulit Event Modeling Tooling](https://nebulit.de/eventmodeling-tooling) to:
- validate event models created in the UI  
- export and import complete models  
- enable automation, prototype generation, and integration with downstream systems

---

## 📘 Schema Overview

The schema describes one top-level object with the following structure:

```json
{
  "slices": [ { ... } ]
}
```

Each **slice** groups related model elements that represent one vertical part of the system (for example: *Submit Cart*, *Fulfill Order*, *Publish Cart*).

### Key Definitions

| Definition | Description |
|-------------|-------------|
| `Slice` | A vertical model slice containing commands, events, read models, and automations |
| `Element` | A single model element (Command, Event, Read Model, Screen, or Automation) |
| `Specification` | A behavior specification describing *given/when/then* examples |
| `Field` | A typed attribute belonging to a command, event, or projection |
| `Dependency` | A link between elements indicating message flow |
| `Actor` | Represents a system user or external system interacting with commands |
| `ScreenImage` | A UI representation of a screen |
| `Table` | A structured view used by projections or read models |

---

## 🧩 Example (Simplified)

A minimal valid JSON document might look like:

```json
{
  "slices": [
    {
      "id": "submit-cart",
      "title": "Submit Cart",
      "status": "Created",
      "context": "Shop",
      "sliceType": "STATE_CHANGE",
      "commands": [
        {
          "id": "submit-cart",
          "title": "Submit Cart",
          "type": "COMMAND",
          "fields": [],
          "dependencies": []
        }
      ],
      "events": [
        {
          "id": "cart-submitted",
          "title": "Cart Submitted",
          "type": "EVENT",
          "fields": [],
          "dependencies": []
        }
      ],
      "readmodels": [],
      "screens": [],
      "processors": [],
      "tables": [],
      "specifications": []
    }
  ]
}
```

You can use this file to **validate** that your event model matches the official structure.

---

## 🔌 Integrating with the Event Modeling Toolkit

The [`openapi.spec.json`](./openapi.spec.json) in this repository describes the local server API used by the [Nebulit Event Modeling Tooling](https://nebulit.de/eventmodeling-tooling).

**Any server running on `http://localhost:3001` that implements this OpenAPI spec will automatically communicate with the Event Modeling Toolkit.**

This means you can:
- Build a custom local backend (Node, Python, Go, etc.) that serves the described endpoints
- Use it to integrate your own storage, git workflow, or code generation pipeline
- The toolkit will connect to it seamlessly — no additional configuration required

Key endpoints the toolkit relies on:

| Endpoint | Purpose |
|----------|---------|
| `GET /api/ping` | Health check / connection test |
| `GET /api/slices` | Load slice data at a given git revision |
| `POST /api/generate` | Persist slice and config files to disk |
| `GET /api/slice-info` | Retrieve slices with status and specification metadata |
| `GET /api/config` | Read server/repo configuration |
| `GET /api/git` | Check git repository status |
| `POST /api/git` | Stage and commit slice changes |

---

## 🧰 Usage

### Validate a model file

You can validate a model JSON file against this schema using any JSON Schema validator.

#### Using `ajv-cli`

```bash
npm install -g ajv-cli
ajv validate -s eventmodeling.schema.json -d my-model.json
```

If your model file is valid, `ajv` will print:

```
my-model.json valid
```

Otherwise, it will report structural issues.

---

## 🧪 Integration with Nebulit Tooling

This schema is directly integrated with the [Nebulit Event Modeling Tooling](https://nebulit.de/eventmodeling-tooling), which provides:

- A visual modeling environment  
- Automatic JSON validation  
- Export/import of models conforming to this schema  
- Live prototype generation  
- Model consistency and dependency checks  

You can use this schema as the foundation for:
- Custom event modeling editors  
- Automated tests validating your event models  
- Code generators for events, commands, and projections  

---

## 🧭 Compatibility

- **JSON Schema Draft:** 07  
- **File extension:** `.json`  
- **Validation tools tested:** `ajv`, `jsonschema`, `python-jsonschema`

---

## 📄 License

This schema is licensed under the **MIT License**.  
You are free to use, adapt, and distribute it in your own tools and workflows.

---

## 🙌 Contributing

Contributions and feedback are welcome.  
If you use this schema in your tooling, integrations, or visual editors, feel free to open a Pull Request or share your use case.

---

## 🔗 Resources

- **Official Tooling:** [https://nebulit.de/eventmodeling-tooling](https://nebulit.de/eventmodeling-tooling)  
- **Event Modeling Website:** [https://eventmodeling.org](https://eventmodeling.org)  
- **JSON Schema Reference:** [https://json-schema.org/specification.html](https://json-schema.org/specification.html)

---

Made with ❤️ to make Event Modeling interoperable and verifiable.

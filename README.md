# CrisisSync-G12

Mini-project for the Engineering Methods course - Group 12: emergency-management system **CrisisSync**.
Desktop client–server application in Java (JavaFX, OCSF over TCP/IP, SQL database with an ORM layer).
This repository holds all assignments and the semester project. The engineering artifacts are kept as text files, so their history is tracked in Git.

## Team

| Name | GitHub |
|---|---|
| Or Elimelech | _(username)_ |
| Eyal Hirsch | _(username)_ |
| Liron Skverchak | _(username)_ |

## Repository structure

```
CrisisSync-G12/
├── README.md
├── .gitignore
└── Assignment1/
    ├── REQUIREMENTS.md        FR / NFR, traceability matrix (RTM), BDD acceptance tests
    ├── G12_Assignment1.vpp    Use case model – Visual Paradigm 18.1
    ├── usecase.puml           PlantUML text of the use case diagram
    └── UC_SPECIFICATIONS.md   Use case specifications (text copy of the model)
```

## Assignment 1 – Use Case Model and Requirements

| File | Content |
|---|---|
| `Assignment1/G12_Assignment1.vpp` | Use case diagram and specifications. Open in Visual Paradigm 18.1; specifications are in *Use Case Details* (Details, Flow of Events). |
| `Assignment1/usecase.puml` | The same diagram as PlantUML: 8 actors, 33 use cases, 28 associations, 5 generalizations, 8 `<<include>>` and 4 `<<extend>>` relationships. |
| `Assignment1/UC_SPECIFICATIONS.md` | Preconditions, postconditions, main flow and extensions of every use case, plus extension points and conditions. |
| `Assignment1/REQUIREMENTS.md` | Requirements, RTM and Given-When-Then acceptance tests for (1) resource bundle reservation including task completion and (2) the SLA report. |

Use case IDs (`UC-xx`), requirement IDs (`FR-…`, `NFR-…`) and test IDs (`AT-…`) are identical across all files.

## Working rules

- Every team member commits regularly (at least once a week) from their own GitHub account, on the `main` branch.
- The same repository is used for all assignments; each assignment gets its own folder.

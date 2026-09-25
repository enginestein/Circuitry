Power Button -> 

```mermaid
flowchart TD
    A[Start] --> B[Input Data]
    B --> C{Valid?}
    C -->|Yes| D[Process Data]
    C -->|No| E[Show Error]
    D --> F[Generate Output]
    E --> B
    F --> G[End]
```
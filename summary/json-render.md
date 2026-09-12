---
source_url: https://json-render.dev/
fetched: 2026-05-14
type: project homepage
topics:
  - developer-tools
---

# json-render: The Generative UI Framework

## Overview

json-render is an open-source framework created by Vercel Labs that enables developers to generate dynamic, personalized user interfaces from AI prompts. The framework connects AI output to UI rendering through a JSON intermediate format, allowing safe and predictable component generation.

## Core Concept

The workflow follows three steps:
1. **Define Your Catalog** – Set guardrails by specifying which components, actions, and data bindings AI can utilize
2. **AI Generates** – AI creates JSON output constrained to the predefined catalog
3. **Render Instantly** – Components render progressively as JSON streams arrive

## Key Features

- **Generative UI**: Create dynamic interfaces from natural language prompts
- **Guardrails**: AI output restricted to developer-defined catalogs
- **Streaming**: Progressive rendering as JSON data arrives
- **Multi-platform**: Support for React and React Native
- **Data Binding**: Connect props to state using $state, $item, $index with two-way binding
- **Code Export**: Export generated UIs as standalone React code with no runtime dependencies

## Available Components (41 Total)

The framework includes 41 pre-built components across layout, input, display, and data visualization categories:

**Layout**: Stack, Grid, Card, Carousel, Accordion, Collapsible, Tabs, Dialog, Drawer

**Input**: Input, Textarea, Select, Checkbox, Radio, Toggle, Switch, Button, ButtonGroup, ToggleGroup, Slider, DropdownMenu

**Display**: Text, Heading, Badge, Avatar, Icon, Image, Metric, Progress, Rating, Separator, Skeleton, Spinner, Tooltip, Popover, Link

**Data Visualization**: BarGraph, LineGraph, Table

**Navigation**: Pagination

**Feedback**: Alert

## Schema Definition Example

```javascript
import { defineSchema, defineCatalog } from '@json-render/core';
import { z } from 'zod';

const schema = defineSchema({ /* ... */ });

export const catalog = defineCatalog(schema, {
  components: {
    Card: {
      props: z.object({
        title: z.string(),
        description: z.string().nullable(),
      }),
      hasChildren: true,
    },
    Metric: {
      props: z.object({
        label: z.string(),
        statePath: z.string(),
        format: z.enum(['currency', 'percent']),
      }),
    },
  },
  actions: {
    export: { params: z.object({ format: z.string() }) },
  },
});
```

## Generated JSON Structure Example

```json
{
  "root": "dashboard",
  "elements": {
    "dashboard": {
      "type": "Card",
      "props": {
        "title": "Revenue Dashboard"
      },
      "children": ["revenue"]
    },
    "revenue": {
      "type": "Metric",
      "props": {
        "label": "Total Revenue",
        "statePath": "/metrics/revenue",
        "format": "currency"
      }
    }
  }
}
```

## Code Export Example

The framework can export to standalone React code:

```javascript
"use client";

import { Card, Metric, Chart } from "@/components/ui";

const data = {
  analytics: {
    revenue: 125000,
    salesByRegion: [
      { label: "US", value: 45000 },
      { label: "EU", value: 35000 },
    ],
  },
};

export default function Page() {
  return (
    <Card data={data} title="Revenue">
      <Metric
        data={data}
        label="Total Revenue"
        statePath="analytics/revenue"
        format="currency"
      />
      <Chart data={data} statePath="analytics/salesByRegion" />
    </Card>
  );
}
```

The exported project includes `package.json`, component files, styles, and all dependencies for independent operation.

## Installation

```
npm install @json-render/core @json-render/react
```

## Resources

- **GitHub**: 15k stars on the repository
- **Playground**: Interactive testing environment available
- **Documentation**: Comprehensive docs with FAQs covering installation, streaming mechanics, available components, and schema creation
- **Examples**: Pre-built example implementations

## Actions Support

The framework includes 6 predefined actions for handling common UI interactions and data operations.

## Creator

Developed by Vercel Labs with the tagline "Made with love by Vercel."

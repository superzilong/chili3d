# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Chili3D is a browser-based 3D CAD application built with TypeScript that achieves near-native performance by compiling OpenCascade (OCCT) to WebAssembly and integrating with Three.js. It's organized as a monorepo with multiple packages under `packages/`.

## Development Commands

### Daily Development
```bash
npm install              # Install dependencies for all workspace packages
npm run dev              # Start dev server at http://localhost:8080
npm run build            # Build the application for production
npm run test             # Run all tests with Jest
npm run testc            # Run tests with coverage
```

### Formatting
```bash
npm run format           # Format TypeScript/JavaScript files (Prettier) AND C++ files (clang-format)
```

### WebAssembly Development
```bash
npm run setup:wasm       # Clone OCCT and install all WebAssembly dependencies (first-time or after updates)
npm run build:wasm       # Build the WebAssembly module (outputs to packages/chili-wasm/lib)
```

The WebAssembly module is built from C++ code in `cpp/` using CMake and Emscripten. It compiles OpenCascade (OCCT) to WebAssembly for CAD kernel functionality.

### Testing

Tests are located in `packages/*/test/` directories and use Jest with ts-jest. Test files follow the pattern `*.test.ts`. The test environment is configured for jsdom to support DOM APIs.

To run tests for specific packages:
```bash
# Example: run tests matching a pattern
node --experimental-vm-modules node_modules/jest/bin/jest.js packages/chili-core/test/
```

## Architecture

### Package Structure

The codebase is organized as a monorepo with the following packages (dependency order from bottom to top):

**Core Layer:**
- `chili-core`: Foundation layer providing interfaces and base classes for the entire application
  - Application, Document, Command patterns
  - Model (INode, Component)
  - Foundation utilities (Observable, History, Transactions)
  - Visual and Shape interfaces
  - Math utilities (XYZ, Matrix, Plane)
  - UI interfaces (IWindow)
  - Serialization framework
  - Service infrastructure

**Geometry & Visualization:**
- `chili-geo`: Geometry abstractions (depends on chili-core)
- `chili-vis`: Visualization abstractions (depends on chili-core)
- `chili-wasm`: OpenCascade WebAssembly bindings and shape factory implementation
  - Implements IShapeFactory from chili-core
  - Wraps OCCT geometry operations
  - Built from C++ code in `cpp/`
- `chili-three`: Three.js-based visual factory implementation
  - Implements IVisualFactory from chili-core
  - Handles 3D rendering, camera controls, visual representations

**UI & Controls:**
- `chili-controls`: UI control components (depends on chili-core)
- `chili-ui`: Main UI implementation including MainWindow and Ribbon interface
  - Depends on chili-core, chili-controls, chili-geo, chili-vis

**Application Layer:**
- `chili-storage`: Document persistence using IndexedDB (depends on chili-core)
- `chili`: Main application package with CAD functionality
  - Commands (create, modify, measure, application)
  - Snapping system (snap handlers, tracking)
  - STEP file import/export
  - Document implementation
  - Services (CommandService, EditorService, HotkeyService)
  - Depends on chili-core, chili-geo, chili-vis

**Builder & Entry:**
- `chili-builder`: Application builder and composition root
  - AppBuilder class for configuring and initializing the application
  - Default ribbon UI configuration
  - Data exchange implementations
  - Depends on chili, chili-three, chili-ui, chili-wasm
- `chili-web`: Entry point for the web application (depends on chili-builder)

### Key Architectural Patterns

**Dependency Injection:**
The application uses a builder pattern (`AppBuilder`) to compose and configure dependencies. Services are registered and started during application initialization.

**Observable & Events:**
The codebase extensively uses the Observer pattern. Most core classes extend `Observable` and use `PropertyChanged` events. A `PubSub` singleton handles global events (e.g., "activeViewChanged").

**Command Pattern:**
All user actions are implemented as commands that extend `CancelableCommand`. Commands implement `ICommand` with an `execute()` method and can be cancelled. Commands are managed by `CommandService`.

**Document-View Architecture:**
- `IApplication`: Singleton managing documents, views, services
- `IDocument`: Represents a CAD document with history, selection, visual representation
- `IView`: Represents a viewport into a document
- Each document has a node hierarchy (`INode`) and transaction-based history

**Serialization:**
Documents and components implement `ISerialize` with a `serialize()` method that returns a `Serialized` object. The document version is tracked via `__DOCUMENT_VERSION__` global.

**Visual Abstraction:**
The rendering layer is abstracted through `IVisualFactory` (implemented by chili-three). This allows the core CAD logic to be rendering-engine agnostic.

**Shape Abstraction:**
CAD geometry is abstracted through `IShapeFactory` (implemented by chili-wasm). The actual geometric operations use OpenCascade via WebAssembly.

### TypeScript Configuration

- Target: ESNext
- Decorators are enabled (`experimentalDecorators: true`)
- Strict mode is enabled
- CSS modules are supported via `typescript-plugin-css-modules`
- Source code is in `packages/`, excluding `packages/chili-occ/occ-wasm` and `packages/chili-wasm/lib`

### Build System

The project uses **Rspack** (not Webpack) for bundling:
- Entry point: `packages/chili-web/src/index.ts`
- CSS modules are enabled
- SWC loader handles TypeScript compilation with decorator support
- Assets: `.wasm`, `.cur`, `.jpg` files are handled as assets
- Global defines: `__APP_VERSION__` and `__DOCUMENT_VERSION__` are injected
- Minimization preserves class names and function names (important for serialization)

### License Headers

All source files must include the AGPL-3.0 license header:
```typescript
// Part of the Chili3d Project, under the AGPL-3.0 License.
// See LICENSE file in the project root for full license information.
```

### Coding Conventions

- The codebase uses interfaces extensively (prefixed with `I`)
- Private fields use underscore prefix (`_fieldName`)
- Getters/setters are used for computed or validated properties
- Async operations return Promises
- Disposable resources implement `IDisposable`

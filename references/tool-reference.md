# AutoCAD MCP Tool Reference

## Table of Contents
- [drawing](#drawing) - File and drawing management
- [entity](#entity) - Entity CRUD and modification
- [layer](#layer) - Layer management
- [block](#block) - Block operations
- [annotation](#annotation) - Text, dimensions, leaders
- [pid](#pid) - P&ID operations (CTO library)
- [view](#view) - Viewport and screenshot
- [system](#system) - Server management and LISP execution

---

## drawing

File and drawing management.

| Operation | Parameters | Notes |
|-----------|-----------|-------|
| `create` | `data: {name?}` | Erases all entities + purges current drawing (resets to clean state). Does NOT create a new tab. |
| `open` | `data: {path}` | Open existing .dwg or .dxf |
| `info` | — | Entity count, layers, extents |
| `save` | `data: {path?}` | QSAVE if no path, SAVEAS if path given |
| `save_as_dxf` | `data: {path}` | Export as DXF format |
| `plot_pdf` | `data: {path}` | Plot to PDF (File IPC only) |
| `purge` | — | Remove unused objects |
| `get_variables` | `data: {names: [...]}` | Get system variables by name |
| `undo` | — | Undo last operation |
| `redo` | — | Redo last undone operation |

## entity

Entity creation, querying, and modification.

**Create operations:**

| Operation | Parameters |
|-----------|-----------|
| `create_line` | `x1, y1, x2, y2, layer?` |
| `create_circle` | `data: {cx, cy, radius}, layer?` |
| `create_polyline` | `points: [[x,y],...], data: {closed?}, layer?` |
| `create_rectangle` | `x1, y1, x2, y2, layer?` |
| `create_arc` | `data: {cx, cy, radius, start_angle, end_angle}, layer?` |
| `create_ellipse` | `data: {cx, cy, major_x, major_y, ratio}, layer?` |
| `create_mtext` | `data: {x, y, width, text, height?}, layer?` |
| `create_hatch` | `entity_id, data: {pattern?}` |

**Read operations:**

| Operation | Parameters |
|-----------|-----------|
| `list` | `layer?` |
| `count` | `layer?` |
| `get` | `entity_id` |

**Modify operations:**

| Operation | Parameters |
|-----------|-----------|
| `copy` | `entity_id, data: {dx, dy}` |
| `move` | `entity_id, data: {dx, dy}` |
| `rotate` | `entity_id, data: {cx, cy, angle}` |
| `scale` | `entity_id, data: {cx, cy, factor}` |
| `mirror` | `entity_id, x1, y1, x2, y2` |
| `offset` | `entity_id, data: {distance}` — File IPC only |
| `array` | `entity_id, data: {rows, cols, row_dist, col_dist}` |
| `fillet` | `data: {id1, id2, radius}` — File IPC only |
| `chamfer` | `data: {id1, id2, dist1, dist2}` — File IPC only |
| `erase` | `entity_id` |

## layer

| Operation | Parameters |
|-----------|-----------|
| `list` | — |
| `create` | `data: {name, color?, linetype?}` |
| `set_current` | `data: {name}` |
| `set_properties` | `data: {name, color?, linetype?, lineweight?}` |
| `freeze` | `data: {name}` |
| `thaw` | `data: {name}` |
| `lock` | `data: {name}` |
| `unlock` | `data: {name}` |

## block

| Operation | Parameters |
|-----------|-----------|
| `list` | — |
| `insert` | `data: {name, x, y, scale?, rotation?, block_id?}` |
| `insert_with_attributes` | `data: {name, x, y, scale?, rotation?, attributes: {tag: val}}` |
| `get_attributes` | `data: {entity_id}` |
| `update_attribute` | `data: {entity_id, tag, value}` |
| `define` | `data: {name, entities: [...]}` — ezdxf only |

## annotation

| Operation | Parameters |
|-----------|-----------|
| `create_text` | `data: {x, y, text, height?, rotation?, layer?}` |
| `create_dimension_linear` | `data: {x1, y1, x2, y2, dim_x, dim_y}` |
| `create_dimension_aligned` | `data: {x1, y1, x2, y2, offset}` |
| `create_dimension_angular` | `data: {cx, cy, x1, y1, x2, y2}` |
| `create_dimension_radius` | `data: {cx, cy, radius, angle}` |
| `create_leader` | `data: {points: [[x,y],...], text}` |

## pid

P&ID drawing with CTO symbol library (requires `C:\PIDv4-CTO\`).

| Operation | Parameters |
|-----------|-----------|
| `setup_layers` | — |
| `insert_symbol` | `data: {category, symbol, x, y, scale?, rotation?}` |
| `list_symbols` | `data: {category}` |
| `draw_process_line` | `data: {x1, y1, x2, y2}` |
| `connect_equipment` | `data: {x1, y1, x2, y2}` |
| `add_flow_arrow` | `data: {x, y, rotation?}` |
| `add_equipment_tag` | `data: {x, y, tag, description?}` |
| `add_line_number` | `data: {x, y, line_num, spec}` |
| `insert_valve` | `data: {x, y, valve_type, rotation?}` |
| `insert_instrument` | `data: {x, y, instrument_type, rotation?, tag_id?, range_value?}` |
| `insert_pump` | `data: {x, y, pump_type, rotation?}` |
| `insert_tank` | `data: {x, y, tank_type, scale?}` |

## view

| Operation | Parameters |
|-----------|-----------|
| `zoom_extents` | — |
| `zoom_window` | `x1, y1, x2, y2` |
| `get_screenshot` | — (returns PNG image) |

## system

| Operation | Parameters |
|-----------|-----------|
| `status` | — |
| `health` | — |
| `get_backend` | — |
| `runtime` | — |
| `init` | — (re-initialize backend) |
| `execute_lisp` | `data: {code}` — File IPC only |

### execute_lisp Examples

**Simple expression:**
```
system(operation="execute_lisp", data={"code": "(+ 1 2)"})
→ "3"
```

**Draw a circle:**
```
system(operation="execute_lisp", data={"code": "(progn (command \"_CIRCLE\" (list 10 10 0) 5) \"circle drawn\")"})
```

**Query system variable:**
```
system(operation="execute_lisp", data={"code": "(getvar \"ACADVER\")"})
```

**Multi-command batch:**
```
system(operation="execute_lisp", data={"code": "(progn\n  (setvar \"CLAYER\" \"EQUIPMENT\")\n  (command \"_RECTANG\" (list 0 0 0) (list 10 10 0))\n  (command \"_CIRCLE\" (list 20 5 0) 3)\n  \"done\")"})
```

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `AUTOCAD_MCP_BACKEND` | `auto` | `auto`, `file_ipc`, or `ezdxf` |
| `AUTOCAD_MCP_IPC_DIR` | `C:/temp` | IPC file exchange directory |
| `AUTOCAD_MCP_IPC_TIMEOUT` | `10.0` | Command timeout in seconds (1-300) |
| `AUTOCAD_MCP_ONLY_TEXT` | `false` | Disable screenshot capture |

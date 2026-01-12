# M6 Reporte — 2026-01-12T11:17:01

## Artefactos
- Baseline: `g:\source\VisualCode\repos\olive-python-vscode-labs\models\add_const.onnx`
- Cuantizado (Olive): `g:\source\VisualCode\repos\olive-python-vscode-labs\outputs\m6_workflow\model.onnx`
- Final (ORT offline optimized): `g:\source\VisualCode\repos\olive-python-vscode-labs\outputs\m6_final\add_const_int8_ort_optimized.onnx`

## Métricas

| Variante | Tamaño | Latencia avg (ms) | Max abs diff vs baseline |
|---|---:|---:|---:|
| Baseline | 90.0 B | 0.003854 | 0.0 |
| Cuantizado (Olive) | 406.0 B | 0.004257 | 0.000000 |
| Final (ORT opt) | 406.0 B | 0.003726 | 0.000000 |

## Comparativas (vs baseline)
- Reducción de tamaño (Cuantizado): -351.11%
- Reducción de tamaño (Final): -351.11%
- Mejora de latencia (Cuantizado): -10.44%
- Mejora de latencia (Final): 3.31%

## Nota
- La latencia aquí es un micro‑benchmark simple en Python.
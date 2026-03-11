# leafviz

**Leaf-shaped concentration gradient plots for Sansevieria**

`leafviz` is a small ggplot2 extension that draws *Sansevieria trifasciata* or
*S. cylindrica* leaf silhouettes as plot glyphs, with the leaf fill color encoding
median concentration (white = low, dark green = high). A transparent boxplot
and raw data points are optionally overlaid inside the leaf outline.

---

## Installation

```r
# From a local source build:
install.packages("path/to/leafviz_0.1.0.tar.gz", repos = NULL, type = "source")

# Or once on GitHub:
# remotes::install_github("yourname/leafviz")
```

---

## Quick start

```r
library(leafviz)
library(ggplot2)
library(dplyr)

# Simulate data in the same format as your analysis
df <- expand.grid(
  Leaf_position = factor(c("Bottom","Middle","Top"),
                         levels = c("Bottom","Middle","Top")),
  Species       = c("Sansevieria_trifasciata", "Sansevieria_cylindrica"),
  Condition     = c("Control", "Drought"),
  Time_point    = factor(c("9","3"), levels = c("9","3"))
) %>%
  slice(rep(1:n(), each = 8)) %>%
  mutate(suc = abs(rnorm(n(), mean = 6, sd = 2.5)))

# ── Trifasciata plot ──────────────────────────────────────────
p <- plot_leaf_gradient(
  data       = df,
  y_label    = "Sucrose (µmol g⁻¹ FW)",
  species    = "trifasciata",
  row_labels = c(`9` = "9 AM", `3` = "3 PM"),
  fix_scale  = TRUE
)
print(p)
```

### Using geom_leaf() directly

```r
ggplot(df %>% filter(Species == "Sansevieria_trifasciata",
                     Condition == "Control",
                     Time_point == "9"),
       aes(x = Leaf_position, y = suc)) +
  geom_leaf(
    species        = "trifasciata",
    low_col        = "white",
    high_col       = "#1a7a1a",
    show_boxplot   = TRUE,
    show_beeswarm  = TRUE
  ) +
  theme_bw() +
  labs(x = "Leaf position", y = "Sucrose (µmol g⁻¹ FW)")
```

### Cylindrica variant

```r
ggplot(df %>% filter(Species == "Sansevieria_cylindrica"),
       aes(x = Leaf_position, y = suc)) +
  geom_leaf(species = "cylindrica") +
  theme_bw()
```

### Drop-in replacement for your existing code

Replace your `p_gradient` block with:

```r
library(leafviz)

p_gradient <- plot_leaf_gradient(
  data          = df_gradient,       # your filtered data frame
  x_var         = "Leaf_position",
  y_var         = "suc",
  facet_row     = "Time_point",
  facet_col     = c("Species", "Condition"),
  species       = "trifasciata",     # or "cylindrica"
  fix_scale     = TRUE,
  show_boxplot  = TRUE,
  show_beeswarm = TRUE,
  y_label       = "Sucrose (µmol g⁻¹ FW)",
  row_labels    = c(`9` = "9 AM", `3` = "3 PM"),
  cld_data      = all_cld_gradient   # your Tukey CLD output
)

print(p_gradient)
ggsave("sucrose_leaf_gradient.png", p_gradient,
       width = 12, height = 6, dpi = 150)
```

---

## Parameters

### `geom_leaf()`

| Parameter | Default | Description |
|-----------|---------|-------------|
| `species` | `"trifasciata"` | Leaf silhouette shape: `"trifasciata"` (broad, flat) or `"cylindrica"` (narrow, tubular) |
| `low_col` | `"white"` | Color for the lowest concentration |
| `high_col` | `"#1a7a1a"` | Color for the highest concentration |
| `global_min` | `NULL` | Fix color scale minimum (for multi-facet consistency) |
| `global_max` | `NULL` | Fix color scale maximum |
| `width` | `NULL` | Leaf half-width in panel npc units (auto-set per species) |
| `show_boxplot` | `TRUE` | Draw IQR box + whiskers + median inside leaf |
| `show_beeswarm` | `TRUE` | Draw jittered raw points inside leaf |

### `plot_leaf_gradient()`

All parameters of `geom_leaf()`, plus:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `x_var` | `"Leaf_position"` | Column for x-axis |
| `y_var` | `"suc"` | Column for response |
| `facet_row` | `"Time_point"` | Column for facet rows |
| `facet_col` | `c("Species","Condition")` | Columns for facet columns |
| `fix_scale` | `TRUE` | Use global min/max for color scale |
| `cld_data` | `NULL` | Data frame with Tukey CLD letters |
| `row_labels` | `NULL` | Named vector for row facet labels |

---

## How it works

1. **StatLeaf** computes the median concentration per x-group and maps it to a
   fill color using a white→green ramp.
2. **GeomLeaf** draws the leaf polygon, optionally layering a boxplot and
   jittered points on top, all clipped to the leaf boundary.
3. Leaf shape coordinates are generated analytically — no external image files needed.

---

## Leaf shapes

| Species | Shape characteristics |
|---------|-----------------------|
| `trifasciata` | Broad lanceolate; wide in the lower 2/3, sharp taper to tip |
| `cylindrica` | Narrow, near-constant width; represents the cylindrical cross-section |

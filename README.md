# R-studio-Sansevieria-leaf-gradients
#Created with the help of Claude Sonnet 4.6, the package aims to represent metabolite gradients across single leaf blades. It uses ggplot2 to showcase box plots and these are organized according to their labeled leaf position. For this reason, the original .csv dataframe must contain a column called "leaf position" with either Top, Middle or Bottom.

#Install simply by:

install.packages("path/leafviz_0.3.0.tar.gz", repos = NULL, type = "source") #Substitute "path" with the path to the .tar file. No need to decompress, R automatically handles it. 

library(leafviz) #(load the library)

#Substiutute df_gradient with your dataframe name (R environment dataframe object):

p <- plot_leaf_gradient(
  data = df_gradient, species = "cylindrica", #Species can be trifasciata or cylindrica
  fix_scale = TRUE, unified = TRUE,
  y_label = "Sucrose (μmol g−1 FW)",
  row_labels = c(`9` = "9 AM", `3` = "3 PM"),
  taper_power = 1.5, base_spread = 2,
  box_lwd = 10, median_lwd = 0.4, whisker_lwd = 0.15,
  show_axis_y = TRUE, show_grid_y = TRUE,
  height = 2,
)

print(p)




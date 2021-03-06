# Dynamic-filter-
How to create dynamic filtering in data science reports and dashboards
library(shiny)
library(magrittr)

ships <- read.csv("https://raw.githubusercontent.com/Appsilon/crossfilter-demo/master/app/ships.csv")

ui <- shinyUI(fluidPage(
  fluidRow(
    column(6, leaflet::leafletOutput("map")),
    column(6, DT::dataTableOutput("tbl"))
  )
))

server <- shinyServer(function(input, output) {
  
  output$map <- leaflet::renderLeaflet({
    leaflet::leaflet(ships) %>%
      leaflet::addTiles() %>% 
      leaflet::addMarkers()
  })
  
  in_bounding_box <- function(data, lat, long, bounds) {
    data %>%
      dplyr::filter(lat > bounds$south & lat < bounds$north & long < bounds$east & long > bounds$west)
  }
  
  data_map <- reactive({
    if (is.null(input$map_bounds)){
      ships
    } else {
      bounds <- input$map_bounds
      in_bounding_box(ships, lat, long, bounds)
    }
  })
  
  output$tbl <- DT::renderDataTable({
    DT::datatable(data_map(), extensions = "Scroller", style = "bootstrap", class = "compact", width = "100%",
                  options = list(deferRender = TRUE, scrollY = 300, scroller = TRUE, dom = 'tp'))
  })
  
})

shinyApp(ui = ui, server = server)

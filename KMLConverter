import utm
import simplekml
from tkinter import *
from tkinter import messagebox
from tkinter import filedialog
import os
import webbrowser
from tkhtmlview import HTMLLabel
import configparser

# Ruta del archivo de configuración
config_file = "config.ini"

# Variables globales
hemisferio_config = ""
zona_config = ""

descripcion_kml = ""

def guardar_configuracion():
    # Obtener los valores seleccionados en los menús desplegables
    zona = zona_variable.get()
    hemisferio = hemisferio_variable.get()
    
    # Crear el archivo de configuración
    config = configparser.ConfigParser()
    config["Configuracion"] = {"zona": zona, "hemisferio": hemisferio}
    
    # Guardar la configuración en el archivo
    with open(config_file, "w") as f:
        config.write(f)

    messagebox.showinfo("Información", "La configuración se ha guardado correctamente.")


def cargar_configuracion():
    global zona_config
    global hemisferio_config
    
    # Verificar si el archivo de configuración existe
    if os.path.exists(config_file):
        # Cargar la configuración del archivo
        config = configparser.ConfigParser()
        config.read(config_file)
        
        # Obtener los valores de configuración
        zona_config = config.get("Configuracion", "zona", fallback="")
        hemisferio_config = config.get("Configuracion", "hemisferio", fallback="")
    else:
        # Crear el archivo de configuración con valores predeterminados
        guardar_configuracion("", "")


def cargar_configuracion():
    global zona_config
    global hemisferio_config
    
    # Verificar si el archivo de configuración existe
    if os.path.exists(config_file):
        # Cargar la configuración del archivo
        config = configparser.ConfigParser()
        config.read(config_file)
        
        # Obtener los valores de configuración
        zona_config = config.get("Configuracion", "zona", fallback="")
        hemisferio_config = config.get("Configuracion", "hemisferio", fallback="")

def convertir_coordenadas():
    # Verificar si se ha seleccionado el hemisferio, la zona y se han agregado las coordenadas de latitud y longitud
    if not hemisferio_variable.get() or not zona_variable.get() or not coordenadas_x_text.get(1.0, END).strip() or not coordenadas_y_text.get(1.0, END).strip():
        messagebox.showerror("Error", "Por favor, complete todos los campos.")
        return
    
    # Obtener los valores seleccionados en los menús desplegables
    zona = int(zona_variable.get())
    hemisferio = hemisferio_variable.get()
    
    # Convertir las coordenadas UTM a grados decimales
    coordenadas_decimales = []
    for x, y in zip(coordenadas_x_text.get(1.0, END).strip().split('\n'), coordenadas_y_text.get(1.0, END).strip().split('\n')):
        lat, lon = utm.to_latlon(float(x), float(y), zona, hemisferio)
        coordenadas_decimales.append((lat, lon))
    
    # Guardar las coordenadas en un archivo de texto
    guardar_coordenadas(coordenadas_decimales)

def guardar_coordenadas(coordenadas):
    archivo = filedialog.asksaveasfile(defaultextension=".txt", filetypes=[("Archivos de texto", "*.txt")])
    if archivo:
        for coordenada in coordenadas:
            lat, lon = coordenada
            archivo.write(f"Latitud: {lat}\tLongitud: {lon}\n")
        archivo.close()

def cargar_coordenadas():
    archivo = filedialog.askopenfilename(filetypes=[("Archivos de texto", "*.txt")])
    if archivo:
        coordenadas_decimales = []
        with open(archivo, "r") as f:
            lineas = f.readlines()
            for linea in lineas:
                latitud, longitud = linea.strip().split("\t")
                latitud = float(latitud.split(":")[1])
                longitud = float(longitud.split(":")[1])
                coordenadas_decimales.append((latitud, longitud))
        
        # Actualizar las coordenadas en los campos de texto
        coordenadas_x_text.delete(1.0, END)
        coordenadas_y_text.delete(1.0, END)
        for coordenada in coordenadas_decimales:
            latitud, longitud = coordenada
            coordenadas_x_text.insert(END, f"{utm.from_latlon(latitud, longitud)[0]}\n")
            coordenadas_y_text.insert(END, f"{utm.from_latlon(latitud, longitud)[1]}\n")

def generar_kml():
    # Verificar si se ha seleccionado el hemisferio, la zona y se han agregado las coordenadas de latitud y longitud
    if not hemisferio_variable.get() or not zona_variable.get() or not coordenadas_x_text.get(1.0, END).strip() or not coordenadas_y_text.get(1.0, END).strip():
        messagebox.showerror("Error", "Por favor, complete todos los campos.")
        return
    
    # Obtener las coordenadas convertidas
    coordenadas_decimales = []
    for x, y in zip(coordenadas_x_text.get(1.0, END).strip().split('\n'), coordenadas_y_text.get(1.0, END).strip().split('\n')):
        lat, lon = utm.to_latlon(float(x), float(y), int(zona_variable.get()), hemisferio_variable.get())
        coordenadas_decimales.append((lat, lon))
    
    # Crear un objeto KML
    kml = simplekml.Kml()
    
    # Obtener el tipo de geometría seleccionado
    tipo_geometria = tipo_geometria_variable.get()
    
    # Obtener el nombre del archivo
    archivo_kml = nombre_entry.get() + ".kml"
    
    # Agregar descripción si existe
    if descripcion_kml:
        if agregar_html_kml:
            descripcion = simplekml.CDATA(f"<b>{nombre_entry.get()}</b><br><br>{descripcion_kml}")
        else:
            descripcion = descripcion_kml
        kml.document.newdescription(descripcion)
    
    # Agregar los puntos, polilíneas o polígonos al KML
    nombre = nombre_entry.get()
    if tipo_geometria == "Punto":
        for coordenada in coordenadas_decimales:
            lat, lon = coordenada
            kml.newpoint(name=nombre, coords=[(lon, lat)])
    elif tipo_geometria == "Polilínea":
        linea = kml.newlinestring(name=nombre)
        linea.coords = [(lon, lat) for lat, lon in coordenadas_decimales]
    elif tipo_geometria == "Polígono":
        poligono = kml.newpolygon(name=nombre)
        poligono.outerboundaryis = [(lon, lat) for lat, lon in coordenadas_decimales]
    
    # Guardar el archivo KML en la carpeta del programa
    kml.save(archivo_kml)
    
    mensaje = f"El archivo KML se ha generado correctamente en '{archivo_kml}'."
    messagebox.showinfo("Información", mensaje)

def generar_tabla_html():
    # Verificar si se ha seleccionado el hemisferio, la zona y se han agregado las coordenadas de latitud y longitud
    if not hemisferio_variable.get() or not zona_variable.get() or not coordenadas_x_text.get(1.0, END).strip() or not coordenadas_y_text.get(1.0, END).strip():
        messagebox.showerror("Error", "Por favor, complete todos los campos.")
        return
    
    # Obtener las coordenadas convertidas
    coordenadas_decimales = []
    for x, y in zip(coordenadas_x_text.get(1.0, END).strip().split('\n'), coordenadas_y_text.get(1.0, END).strip().split('\n')):
        lat, lon = utm.to_latlon(float(x), float(y), int(zona_variable.get()), hemisferio_variable.get())
        coordenadas_decimales.append((lat, lon))
    
    # Crear el contenido HTML de la tabla
    contenido_html = f'''
    <html>
    <head>
        <style>
            table, th, td {{
                border: 1px solid black;
                border-collapse: collapse;
            }}
            th, td {{
                padding: 5px;
            }}
        </style>
    </head>
    <body>
        <h2>{nombre_entry.get()}</h2>
        <table>
            <tr>
                <th colspan="2">Coordenadas UTM</th>
            </tr>
            <tr>
                <th>X</th>
                <th>Y</th>
            </tr>
    '''
    for coordenada in coordenadas_decimales:
        lat, lon = coordenada
        x_formateado = "{:.4f}".format(utm.from_latlon(lat, lon)[0])
        y_formateado = "{:.4f}".format(utm.from_latlon(lat, lon)[1])
        contenido_html += f'''
            <tr>
                <td>{x_formateado}</td>
                <td>{y_formateado}</td>
            </tr>
        '''
    
    contenido_html += '''
        </table>
    </body>
    </html>
    '''
    
    # Mostrar el contenido HTML en una ventana
    mostrar_html(archivo_temporal, contenido_html)

def previsualizar_html(archivo):
    # Leer el contenido del archivo HTML
    with open(archivo, "r") as f:
        contenido_html = f.read()
    
    # Crear una ventana de previsualización
    previsualizacion_ventana = Toplevel(root)
    previsualizacion_ventana.title("Previsualización de la tabla HTML")
    
    # Mostrar el contenido HTML en un widget HTMLLabel
    html_label = HTMLLabel(previsualizacion_ventana, html=contenido_html)
    html_label.pack()

    # Botón para copiar la tabla HTML
    copiar_boton = Button(previsualizacion_ventana, text="Copiar Tabla", command=lambda: copiar_contenido(contenido_html))
    copiar_boton.pack()

def mostrar_html(archivo, contenido):
    with open(archivo, 'w') as f:
        f.write(contenido)
    webbrowser.open(archivo)

def mostrar_codigo_html():
    # Leer el contenido del archivo HTML
    with open(archivo_temporal, "r") as f:
        contenido_html = f.read()
    
    # Crear una ventana para mostrar el código HTML
    codigo_html_ventana = Toplevel(root)
    codigo_html_ventana.title("Código HTML")
    
    # Mostrar el código HTML en un widget Text
    codigo_html_text = Text(codigo_html_ventana, height=20, width=80)
    codigo_html_text.pack()
    codigo_html_text.insert(END, contenido_html)
    codigo_html_text.config(state=DISABLED)

    # Botón para copiar el código HTML
    copiar_boton = Button(codigo_html_ventana, text="Copiar Código", command=lambda: copiar_contenido(contenido_html))
    copiar_boton.pack()

def copiar_contenido(contenido):
    root.clipboard_clear()
    root.clipboard_append(contenido)
    messagebox.showinfo("Información", "El contenido se ha copiado al portapapeles.")

def actualizar_nombre_etiqueta():
    tipo_geometria = tipo_geometria_variable.get()
    if tipo_geometria == "Punto":
        nombre_label.config(text="Nombre de los puntos:")
    elif tipo_geometria == "Polilínea":
        nombre_label.config(text="Nombre de la polilínea:")
    elif tipo_geometria == "Polígono":
        nombre_label.config(text="Nombre del polígono:")

def mostrar_propiedades_kml():
    # Crear la ventana de Propiedades KML
    propiedades_ventana = Toplevel(root)
    propiedades_ventana.title("Propiedades KML")

    # Variables para almacenar la selección del usuario
    agregar_html = BooleanVar()
    tamanio_linea = DoubleVar()
    color_linea = StringVar()
    color_poligono = StringVar()

    # Función para guardar la selección y cerrar la ventana
    def guardar_propiedades():
        global agregar_html_kml
        global descripcion_kml
        global tamanio_linea_config
        global color_linea_config
        global color_poligono_config

        agregar_html_kml = agregar_html.get()
        tamanio_linea_config = tamanio_linea.get()
        color_linea_config = color_linea.get()
        color_poligono_config = color_poligono.get()
        propiedades_ventana.destroy()

    # Etiqueta y opción para agregar código HTML
    agregar_html_label = Label(propiedades_ventana, text="Agregar código HTML:")
    agregar_html_label.pack()

    agregar_html_check = Checkbutton(propiedades_ventana, variable=agregar_html)
    agregar_html_check.pack()

    # Etiqueta y opción para el tamaño de la línea
    tamanio_linea_label = Label(propiedades_ventana, text="Tamaño de la línea:")
    tamanio_linea_label.pack()

    tamanio_linea_scale = Scale(propiedades_ventana, variable=tamanio_linea, from_=0.1, to=10, resolution=0.1, orient=HORIZONTAL)
    tamanio_linea_scale.pack()

    # Etiqueta y opción para el color de la línea
    color_linea_label = Label(propiedades_ventana, text="Color de la línea:")
    color_linea_label.pack()

    color_linea_entry = Entry(propiedades_ventana, textvariable=color_linea)
    color_linea_entry.pack()

    # Etiqueta y opción para el color del polígono
    color_poligono_label = Label(propiedades_ventana, text="Color del polígono:")
    color_poligono_label.pack()

    color_poligono_entry = Entry(propiedades_ventana, textvariable=color_poligono)
    color_poligono_entry.pack()

    # Botón para guardar la selección
    guardar_boton = Button(propiedades_ventana, text="Guardar", command=guardar_propiedades)
    guardar_boton.pack()

def salir():
    root.quit()  # Sale de la aplicación

# Crear la ventana principal
root = Tk()
root.title("KMLConverter V.1.5")

# Cargar la configuración
cargar_configuracion()

# Crear el menú superior
menu_superior = Menu(root)
root.config(menu=menu_superior)

# Menú "Archivo"
menu_archivo = Menu(menu_superior, tearoff=0)
menu_superior.add_cascade(label="Archivo", menu=menu_archivo)
menu_archivo.add_command(label="Cargar coordenadas", command=cargar_coordenadas)
menu_archivo.add_separator()
menu_archivo.add_command(label="Guardar Coordenadas", command=convertir_coordenadas)
menu_archivo.add_command(label="Guardar configuración", command=guardar_configuracion)
menu_archivo.add_separator()
menu_archivo.add_command(label="Salir", command=salir)

# Menú "Editor"
menu_editor = Menu(menu_superior, tearoff=0)
menu_superior.add_cascade(label="Editor", menu=menu_editor)
menu_editor.add_command(label="Previsualizar Tabla", command=generar_tabla_html)
menu_editor.add_command(label="Mostrar Código HTML", command=mostrar_codigo_html)
menu_editor.add_separator()
menu_editor.add_command(label="Propiedades KML", command=mostrar_propiedades_kml)

def mostrar_acerca_de():
    # Crear la ventana de Acerca de
    ventana_acerca_de = Toplevel(root)
    ventana_acerca_de.title("Acerca de")

    # Texto de descripción
    descripcion = "Conversor de Coordenadas - Versión: 1.5 \n \n Créditos:\n \n O. Contreras \n ChatGPT \n Ecosoluciones Ambientales \n \n Visita el Repositorio de este Programa"
    
    # Label de descripción
    descripcion_label = Label(ventana_acerca_de, text=descripcion)
    descripcion_label.pack(padx=10, pady=10)

    # Frame para los botones
    frame_botones = Frame(ventana_acerca_de)
    frame_botones.pack(pady=5)

    # Función para abrir el enlace
    def abrir_enlace():
        webbrowser.open("https://github.com/Ecosoluciones-Ambientales/KMLConverter")

    # Botón para abrir el enlace
    enlace_boton = Button(frame_botones, text="Abrir enlace", command=abrir_enlace)
    enlace_boton.pack(side=LEFT, padx=5)

    # Asociar la tecla "Esc" con la función de cierre
    ventana_acerca_de.bind("<Escape>", lambda event: ventana_acerca_de.destroy())

    # Establecer el foco en la ventana de Acerca de
    ventana_acerca_de.focus_set()


def mostrar_help():
    # Crear la ventana de Ayuda
    ventana_ayuda = Toplevel(root)
    ventana_ayuda.title("Ayuda")

    # Cargar el contenido HTML del manual
    with open("manual.html", "r") as f:
        contenido_html = f.read()

    # Mostrar el contenido HTML en un widget HTMLLabel
    html_label = HTMLLabel(ventana_ayuda, html=contenido_html)
    html_label.pack(expand=True, fill=BOTH)

# Menú "Información"
menu_informacion = Menu(menu_superior, tearoff=0)
menu_superior.add_cascade(label="Información", menu=menu_informacion)
menu_informacion.add_command(label="Acerca de", command=mostrar_acerca_de)
menu_informacion.add_command(label="Help", command=mostrar_help)


# Frame para los menús desplegables
frame_menus = Frame(root)
frame_menus.pack()

# Menú desplegable para seleccionar la zona
zona_label = Label(frame_menus, text="Zona:")
zona_label.pack(side=LEFT)

zona_variable = StringVar(root)
zona_variable.set(zona_config)
zona_menu = OptionMenu(frame_menus, zona_variable, *range(1, 61))
zona_menu.pack(side=LEFT)

# Menú desplegable para seleccionar el hemisferio
hemisferio_label = Label(frame_menus, text="Hemisferio:")
hemisferio_label.pack(side=LEFT)

hemisferio_variable = StringVar(root)
hemisferio_variable.set(hemisferio_config)
hemisferio_menu = OptionMenu(frame_menus, hemisferio_variable, "Norte", "Sur")
hemisferio_menu.pack(side=LEFT)

# Etiqueta y entrada para las coordenadas UTM en X
coordenadas_x_label = Label(root, text="Coordenadas UTM en X:")
coordenadas_x_label.pack()

coordenadas_x_text = Text(root, height=5, width=30)
coordenadas_x_text.pack()

# Etiqueta y entrada para las coordenadas UTM en Y
coordenadas_y_label = Label(root, text="Coordenadas UTM en Y:")
coordenadas_y_label.pack()

coordenadas_y_text = Text(root, height=5, width=30)
coordenadas_y_text.pack()

# Checkbox para seleccionar el tipo de geometría
tipo_geometria_label = Label(root, text="Tipo de geometría:")
tipo_geometria_label.pack()

tipo_geometria_variable = StringVar()
tipo_geometria_frame = Frame(root)
tipo_geometria_frame.pack()

tipo_geometria_checkbox1 = Radiobutton(tipo_geometria_frame, text="Punto", variable=tipo_geometria_variable, value="Punto", command=actualizar_nombre_etiqueta)
tipo_geometria_checkbox1.pack(side=LEFT)
tipo_geometria_checkbox2 = Radiobutton(tipo_geometria_frame, text="Polilínea", variable=tipo_geometria_variable, value="Polilínea", command=actualizar_nombre_etiqueta)
tipo_geometria_checkbox2.pack(side=LEFT)
tipo_geometria_checkbox3 = Radiobutton(tipo_geometria_frame, text="Polígono", variable=tipo_geometria_variable, value="Polígono", command=actualizar_nombre_etiqueta)
tipo_geometria_checkbox3.pack(side=LEFT)

# Cuadro de texto para el nombre del polígono, puntos o polilínea
nombre_label = Label(root, text="Nombre del polígono:")
nombre_label.pack()

nombre_entry = Entry(root)
nombre_entry.pack()

# Botón para generar el archivo KML
kml_boton = Button(root, text="Generar KML", command=generar_kml)
kml_boton.pack()

# Botón para previsualizar la tabla HTML
tabla_html_boton = Button(root, text="Previsualizar Tabla HTML", command=generar_tabla_html)
tabla_html_boton.pack()

# Ruta del archivo temporal para mostrar el contenido HTML
archivo_temporal = "temp.html"

root.geometry("300x400")  # Establece el tamaño de la ventana a 800 píxeles de ancho y 600 píxeles de alto

# Mostrar la ventana principal
root.mainloop()

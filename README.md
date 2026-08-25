# Analizador-Pedidos
Este script en Python procesa archivos de pedidos y productos para evaluar qué tan bien está funcionando la logística de entregas.

print("Pregunta 1")
def lista_pedidos(archivo_pedidos):
    pedidos = []
    archivo = open(archivo_pedidos, 'r')
    #Abrimos el archivo en modo lectura
    es_encabezado = True
    for linea in archivo:
        if es_encabezado == True:
            es_encabezado = False
    #Creamos este if para evitar que se lea el encabezado del archivo
        else:
            datos = linea.strip().split(';')
            id_pedido = datos[0]
            costo = int(datos[4])
            pedidos.append([id_pedido, costo])
    #Usamos la misma logica que la tarea pasada para guardar datos en una lista
    archivo.close()
    return pedidos
print(lista_pedidos('pedidos.csv'))
print('')
#Ahora pasamos a la pregunta 2
print('Pregunta 2')
def desempeño(archivo_pedidos, archivo_productos):

    info_pedidos = lista_pedidos(archivo_pedidos)
    suma_precios = 0
    i = 0
    while i < len(info_pedidos):
        linea = info_pedidos[i]
        suma_precios = suma_precios + int(linea[1])
        i = i + 1
    promedio_costos = suma_precios / len(info_pedidos)
    #Usamos la funcion pasada para sacar el promedio de los precios de los pedidos

    productos = {}
    archivo_prod = open(archivo_productos, 'r')
    es_encabezado = True
    for linea in archivo_prod:
        if es_encabezado == True:
            es_encabezado = False
    #Hacemos lo mismo que la pregunta anterior para saltar el encabezado 
        else:
            datos = linea.strip().split(';')
            id_prod = datos[0]
            nombre = datos[1]
            categoria = datos[2]
            precio_base = int(datos[3])
            productos[id_prod] = [nombre, categoria, precio_base]
    archivo_prod.close()
    #El diccionario creado tiene la siguiente forma: llave = id producto, valor = datos del producto en una lista
    diccionario = {'Eficiente': [], 'Problemático': [], 'Costoso': []}
    archivo_ped = open(archivo_pedidos, 'r')
    es_encabezado = True
    for linea in archivo_ped:
        if es_encabezado == True:
            es_encabezado = False
        else:
            datos = linea.strip().split(';')
            tiempo_estimado = int(datos[2])
            tiempo_real = int(datos[3])
            costo = int(datos[4])
            reclamo = int(datos[5])
            id_prod = datos[6]
        
            info = productos[id_prod]
            nombre_prod = info[0]
            categoria_prod = info[1]
            precio_base = info[2]
            registro = [costo, nombre_prod, tiempo_estimado, tiempo_real, categoria_prod]
        #Definimos todo para colocar cada pedido en los criterios 
            retraso = False
            if tiempo_real > tiempo_estimado:
                retraso = True
            
            #Usamos un if para que cada pedido este en un criterios
            if retraso == False and reclamo == 0 and costo <= promedio_costos:
                diccionario['Eficiente'].append(registro)
                
            if retraso == True and reclamo == 1:
                diccionario['Problemático'].append(registro)
                
            if costo > promedio_costos or costo > precio_base:
                diccionario['Costoso'].append(registro)
                
    archivo_ped.close()
    
    diccionario['Eficiente'].sort()
    diccionario['Problemático'].sort()
    diccionario['Costoso'].sort()
    #Ordenamos el diccionario de menor a mayor
    return diccionario
print(desempeño('pedidos.csv','productos.csv'))
print('')
#Ahora pasamos a la pregunta 3
print('Pregunta 3')
def clasificar(archivo_pedidos, archivo_productos, categoria):
    diccionario = desempeño(archivo_pedidos, archivo_productos)

    criterios = ['Eficiente', 'Problemático', 'Costoso']
    total_pedidos = 0
    
    k = 0
    while k < len(criterios):
        crit_actual = criterios[k]
        total_pedidos = total_pedidos + len(diccionario[crit_actual])
        k = k + 1
    #Veemos la cantidad total de pedidos y la almacenamos para despues imprimirla

    i = 0
    while i < len(criterios):
        crit = criterios[i]
        archivo_nuevo = open(crit + ".txt", 'w')
    #Creamos el archivo con el nombre del criterio y en modo esribir
        lista_pedidos_criterio = diccionario[crit]
        j = 0
        contador = 1
        
        while j < len(lista_pedidos_criterio) and contador <= 10:
        #Contador hasta 10 porque necesitamos hacer solamente un top 10 en cada criterio
            pedido = lista_pedidos_criterio[j]
            costo_ped = pedido[0]
            nombre_ped = pedido[1]
            t_est = pedido[2]
            t_real = pedido[3]
            cat_ped = pedido[4]
            #Definimos los valores para usarlo en una plantilla para escribir en el archivo

            if cat_ped != categoria:
                plantilla = "#{} pedido {}: tpo estimado {} min, tpo real {} min, costo ${}.\n"
            #Usamos esta plantilla para escribir en el archivo
                linea = plantilla.format(str(contador), nombre_ped, str(t_est), str(t_real), str(costo_ped))
                archivo_nuevo.write(linea)
            #Escribimos en el archivo creado con el nombre del criterio
                contador = contador + 1
                
            j = j + 1
            
        archivo_nuevo.close()
        i = i + 1
        
    return total_pedidos
print(clasificar('pedidos_grande.csv','productos_grande.csv','Tecnologia'))

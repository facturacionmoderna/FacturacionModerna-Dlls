FacturacionModerna-Dlls
=======================

Dlls compiladas para diferentes versiones de .Net


## Características

* Comprobante.dll
  * Genera la cadena original
  * Genera el sello del comprobante
  * Obtiene el certificado
  * Agrega el sello al xml

* WSConecFM.dll
  * Crea la conexión con los parámetros ingresados
  * Timbra un xml o un layout
  * Cancela un comprobante


## Ejemplo de uso

1.- Descargar el contenido de las DLLs
 
2.- Implementar la DLL de acuerdo a su versión Framework.Net

3.- Agregar la referencia de la DDL al proyecto

Implementación:
```C#
/* Importar la clase. */
Imports WSConecFM
 
Public Class Form1
 
    Private Sub Button1_Click(sender As Object, e As EventArgs) Handles Button1.Click
        Dim r_wsconect As New WSConecFM.Resultados()
        /*  CREAR LA CONFIGURACION DE CONEXION CON EL SERVICIO SOAP
          * *    Los parametros configurables son:
          * *    1.- string UserID; Nombre de usuario que se utiliza para la conexion con SOAP
          * *    2.- string UserPass; Contraseña del usuario para conectarse a SOAP
          * *    3.- string emisorRFC; RFC del contribuyente
          * *    4.- Boolean generarCBB; Indica si se desea generar el CBB
          * *    5.- Boolean generarTXT; Indica si se desea generar el TXT
          * *    6.- Boolean generarPDF; Indica si se desea generar el PDF
          * *    7.- string urlTimbrado; URL de la conexion con SOAP
          * * La configuración inicial es para el ambiente de pruebas
        */
        
        /* Crear la instancia de la clase requestTimbrarCFDI() para poder cambiar los parametros de conexión */          
        Dim reqt As New requestTimbrarCFDI()
        /* Ejemplo para modifcar el RFC emisor */
        /* reqt.emisorRFC = XXXXXXXXXXXX */
 
        /* Crear instancia de la clase Timbrado() para la generacion de comprobantes */ 
        Dim timbra As New Timbrado()
        Dim xmlfile As String
        
        /* En este ejemplo se esta utilizando un archivo XML para ser timbrado, 
          * * tambien soporta pasarle como parametro el XML en forma de cadena
        */
        xmlfile = "\ruta\hacia\el\archivo\archivo.xml"
        r_wsconect = timbra.Timbrar(xmlfile, reqt)
        If Not r_wsconect.status Then
            MessageBox.Show(r_wsconect.message)
            Close()
        End If
        /*
          * * Código donde se procesas la respuesta r_wsconect, se extrae el XML y el PDF
        */
    End Sub
End Class
```

## Ejemplo de uso
Si tiene alguna duda sobre la implementación de está clase, puede contactarnos a: 

desarrollo@facturacionmoderna.com 

[1]: http://www.facturacionmoderna.com

FacturacionModerna-Dlls
=======================

Dlls compiladas para las versiones 2, 3, 3.5, 4 y 4.5 de Framework.Net


## Características

* Comprobante.dll
  * Creación de la cadena original del xml
  * Creación del sello del comprobante
  * Obtiene información del certificado
  * Agrega el sello, certificado y número de certificado al xml

* WSConecFM.dll
  * Crea la conexión con los parámetros ingresados
  * Timbra un xml o un layout
  * Cancela un comprobante
  * Activa la cancelación con el certificado especificado


## Ejemplo de implementación Comprobante.dll

1.- Descargar el contenido de las DLLs
 
2.- Implementar la DLL de acuerdo a su versión Framework.Net

3.- Agregar la referencia de la DDL al proyecto

Implementación:
```C#
// Importar la clase.
Using Comprobante;
 
private void cmdGenerarSello_Click(object sender, EventArgs e)
{
    // Especificar ruta de los archivos .cer y .key
    string currentPath = Directory.GetParent(Directory.GetCurrentDirectory()).Parent.Parent.Parent.FullName;
    string keyfile = currentPath + "\\utilerias\\certificados\\20001000000200000278.key";
    string certfile = currentPath + "\\utilerias\\certificados\\20001000000200000278.cer";
    
    // Especificar ruta del xslt 
    string xsltPath;
    // Para xml de retenciones:
    xsltPath = currentPath + "\\utilerias\\xslt_retenciones\\retenciones.xslt";
    // Para xml de facturas:
    xsltPath = currentPath + "\\utilerias\\xslt3_2\\cadenaoriginal_3_2.xslt";
    
    // Especificar la contraseña del archivo .key
    string password = "12345678a";
    
    // Especificar la ruta del archivo o bien asignar la cadena de un xml, ejemplos
    // string xml = "C:\ejemploXML.xml";
    // string xml = "<?xml version=\"1.0\" encoding=\"UTF-8\"?>.....lo demas del xml....."
    string xmlfile = xml
    
    /* Crear instancia al objeto comprobante */
    Comprobante.Utilidades obj = new Comprobante.Utilidades();

    /*  OBTENER LA INFORMACION DEL CERTIFICADO
     * *    Los parametros enviados son:
     * *    1.- Ruta del certificado
    */
    string cert_b64 = "";
    string cert_No = "";
    if (obj.getInfoCertificate(certfile))
    {
       cert_b64 = obj.getCertificate();
       cert_No = obj.getCertificateNumber();
    }
    else
    {
       MessageBox.Show(obj.getMessage());
       Environment.Exit(-1);
    }

    /*  AGREGAR INFORMACION DEL CERTIFICADO AL XML ANTES DE GENERAR LA CADENA ORIGINA
     * *    Los parametros enviados son:
     * *    1.- Xml (Puede ser una cadena o una ruta)
     * *    2.- Certificado codificado en base 64
     * *    3.- Numero de certificado
     * Retorna el XML Modificado
    */
    string newXml = obj.addDigitalStamp(xmlfile, cert_b64, cert_No);
    if (newXml.Equals(""))
    {
       MessageBox.Show(obj.getMessage());
       Environment.Exit(-1);
    }
    xmlfile = newXml;

    /* GENERAR CADENA ORIGINAL
     * *   Los paramteros enviado son:
     * *    1.- xml (Puede ser una cadena o una ruta)
     * *    2.- xslt (Ruta del archivo xslt, con el cual se construye la cadena original)
     * *   Retorna la cadena original
    */

    string cadenaO = obj.createOriginalChain(xmlfile, xsltPath);
    if (cadenaO.Equals(""))
    {
       MessageBox.Show(obj.getMessage());
       Environment.Exit(-1);
    }

    /* GENERAR EL SELLO DEL COMPROBANTE
     * *    Los parametros enviado son:
     * *    1.- archivo de llave privada (.key)
     * *    2.- Contraseña del archivo de llave privada
     * *    3.- Cadena Original (Puede ser una cadena o una ruta)
     * Retorna el sello en r_comprobante.message
    */
    string sello = obj.createDigitalStamp(keyfile, password, cadenaO);
    if (sello.Equals(""))
    {
       MessageBox.Show(obj.getMessage());
       Environment.Exit(-1);
    }

    /*  AGREGAR LA INFORMACION DEL SELLO AL XML
     * *    Los parametros enviados son:
     * *    1.- Xml (Puede ser una cadena o una ruta)
     * *    2.- Sello del comprobante
     * Retorna el XML Modificado
    */
    newXml = obj.addDigitalStamp(xmlfile, sello);
    if (newXml.Equals(""))
    {
       MessageBox.Show(obj.getMessage());
       Environment.Exit(-1);
    }

    // Obtenemos un xml con sello e información del certificado, listo para ser timbrado
    // haciendo uso de WSConecFM.dll
    xmlfile = newXML;
}
```


## Ejemplo de implementación WSConecFM.dll

1.- Descargar el contenido de las DLLs
 
2.- Implementar la DLL de acuerdo a su versión Framework.Net

3.- Agregar la referencia de la DDL al proyecto

Implementación:
```C#
// Importar la clase.
Using WSConecFM;
 
private void cmdTimbrarXML_Click(object sender, EventArgs e)
{
   // Especificar una ruta de salida para los archivos XML, PDF, TXT y CBB (Opcional)
   string currentPath = Directory.GetParent(Directory.GetCurrentDirectory()).Parent.Parent.Parent.FullName;
   string resultPath = currentPath + "\\resultados";
   
   /*  CREAR LA CONFIGURACION DE CONEXION CON EL SERVICIO SOAP
    * *    Los parametros configurables son:
    * *    1.- string UserID; Nombre de usuario que se utiliza para la conexion con SOAP
    * *    2.- string UserPass; Contraseña del usuario para conectarse a SOAP
    * *    3.- string emisorRFC; RFC del contribuyente
    * *    4.- Boolean generarCBB; Indica si se desea generar el CBB
    * *    5.- Boolean generarTXT; Indica si se desea generar el TXT
    * *    6.- Boolean generarPDF; Indica si se desea generar el PDF
    * *    7.- string urlTimbrado; URL de la conexion con SOAP
    * La configuracion inicial es para el ambiente de pruebas
   */
   
   WSConecFM.Resultados r_wsconect = new WSConecFM.Resultados();
   requestTimbrarCFDI reqt = new requestTimbrarCFDI();
   /*
    * Si desea cambiar alguna configuracion, solo realizar lo siguiente
    * reqt.generarPDF = true;  Por poner un ejemplo
   */
   
   /*  TIMBRAR XML
    * *    Los parametros enviados son:
    * *    1.- XML; (Acepta una ruta o una cadena)
    * *    2.- Objeto con las configuraciones de conexion con SOAP
    * Retorna un objeto con los siguientes valores codificado en base 64:
    * *    1.- xml en base 64
    * *    2.- pdf en base 64
    * *    3.- png en base 64
    * *    4.- txt en base 64
    * Los valores de retorno van a depender de la configuracion enviada a la función
   */
   
   string xmlfile = "C:\ejemploXML.xml";
   Timbrado timbra = new Timbrado();
   r_wsconect = timbra.Timbrar(xmlfile, reqt);
   if (!r_wsconect.status)
   {
       MessageBox.Show(r_wsconect.message);
       Environment.Exit(-1);
   }
   byte[] byteXML = System.Convert.FromBase64String(r_wsconect.xmlBase64);
   System.IO.FileStream swxml = new System.IO.FileStream((resultPath + ("\\" + (r_wsconect.uuid + ".xml"))), System.IO.FileMode.Create);
   swxml.Write(byteXML, 0, byteXML.Length);
   swxml.Close();
   if (reqt.generarCBB) {
       byte[] byteCBB = System.Convert.FromBase64String(r_wsconect.cbbBase64);
       System.IO.FileStream swcbb = new System.IO.FileStream((resultPath + ("\\" + (r_wsconect.uuid + ".png"))), System.IO.FileMode.Create);
       swcbb.Write(byteCBB, 0, byteCBB.Length);
       swcbb.Close();
   }
   if (reqt.generarPDF)
   {
       byte[] bytePDF = System.Convert.FromBase64String(r_wsconect.pdfBase64);
       System.IO.FileStream swpdf = new System.IO.FileStream((resultPath + ("\\" + (r_wsconect.uuid + ".pdf"))), System.IO.FileMode.Create);
       swpdf.Write(bytePDF, 0, bytePDF.Length);
       swpdf.Close();
   }
   if (reqt.generarTXT)
   {
       byte[] byteTXT = System.Convert.FromBase64String(r_wsconect.txtBase64);
       System.IO.FileStream swtxt = new System.IO.FileStream((resultPath + ("\\" + (r_wsconect.uuid + ".txt"))), System.IO.FileMode.Create);
       swtxt.Write(byteTXT, 0, byteTXT.Length);
       swtxt.Close();
   }
}
```
## Dudas
Si tiene alguna duda sobre la implementación de está clase, puede contactarnos a: 

desarrollo@facturacionmoderna.com 

[1]: https://github.com/facturacionmoderna/FacturacionModerna-CSharp/blob/master/TimbradoCancelado/TimbradoCancelado/frmEjemplos.cs

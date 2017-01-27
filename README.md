FacturacionModerna-Dlls
=======================

Dlls compiladas para las versiones 2, 3, 3.5, 4 y 4.5 de Framework.Net


## Características

* Comprobante.dll
  * Creación de la cadena original del xml
  * Creación del sello del comprobante
  * Obtiene información del certificado
  * Agrega el sello, certificado y número de certificado al xml

* ConnectionWSFM.dll
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
    // haciendo uso de ConnectionWSFM.dll
    xmlfile = newXML;
}
```


## Ejemplo de implementación ConnectionWSFM.dll

1.- Descargar el contenido de las DLLs
 
2.- Implementar la DLL de acuerdo a su versión Framework.Net

3.- Agregar la referencia de la DDL al proyecto

Implementación:
```C#
// Importar la clase.
Using ConnectionWSFM;
 
private void cmdTimbrarXML_Click(object sender, EventArgs e)
{
   // Especificar una ruta de salida para los archivos XML, PDF, TXT y CBB (Opcional)
   string currentPath = Directory.GetParent(Directory.GetCurrentDirectory()).Parent.Parent.Parent.FullName;
   string resultPath = currentPath + "\\resultados";
   
   /*  CREAR LA CONFIGURACION DE CONEXION CON EL SERVICIO SOAP
	 * *    Los parametros configurables son:
	 * *    1.- Nombre de usuario que se utiliza para la conexion al Web Service
	 * *    2.- Contraseña del usuario que se utiliza para la conexion al Web Service
	 * *    3.- RFC Emisor
	 * *    4.- Habilitar el retorno del CBB
	 * *    5.- Habilitar el retorno del TXT
	 * *    6.- Habilitar el retorno del PDF
	 * *    7.- URL del Web Service (endpoint)
	 * *    8.- Habilitar debug para guardar Request y Response (Si se habilita, se debe de especificar una ruta del archivo log)
	 * * La configuracion inicial es para el ambiente de pruebas
	*/
	ConnectionFM conX = new ConnectionFM();
	conX.setDebugMode(true);
	conX.setLogFilePath(currentPath + "\\logs\\log.txt");
	conX.setGenerarPdf(true);


	/*  Timbrar Layout
	 * *   Se envia el layout a timbrar, puede ser una xml o un txt, especificando la ruta del archivo
	 * *   o un string conteniendo todo el layout
	 */
	if (conX.timbrarLayout(newXml) == true)
	{
		byte[] byteXML = System.Convert.FromBase64String(conX.getXmlB64());
		System.IO.FileStream swxml = new System.IO.FileStream((resultPath + ("\\" + (conX.getUuid() + ".xml"))), System.IO.FileMode.Create);
		swxml.Write(byteXML, 0, byteXML.Length);
		swxml.Close();

		if (conX.getCbbB64() != "")
		{
			byte[] byteCBB = System.Convert.FromBase64String(conX.getCbbB64());
			System.IO.FileStream swcbb = new System.IO.FileStream((resultPath + ("\\" + (conX.getUuid() + ".png"))), System.IO.FileMode.Create);
			swcbb.Write(byteCBB, 0, byteCBB.Length);
			swcbb.Close();
		}
		if (conX.getPdfB64() != "")
		{
			byte[] bytePDF = System.Convert.FromBase64String(conX.getPdfB64());
			System.IO.FileStream swpdf = new System.IO.FileStream((resultPath + ("\\" + (conX.getUuid() + ".pdf"))), System.IO.FileMode.Create);
			swpdf.Write(bytePDF, 0, bytePDF.Length);
			swpdf.Close();
		}
		if (conX.getTxtB64() != "")
		{
			byte[] byteTXT = System.Convert.FromBase64String(conX.getTxtB64());
			System.IO.FileStream swtxt = new System.IO.FileStream((resultPath + ("\\" + (conX.getUuid() + ".txt"))), System.IO.FileMode.Create);
			swtxt.Write(byteTXT, 0, byteTXT.Length);
			swtxt.Close();
		}
		MessageBox.Show("Comprobante guardado en " + resultPath + "\\");
	}
	else
	{
		MessageBox.Show("[" + conX.getErrorCode() + "] " + conX.getErrorMessage());
	}

}
```
## Dudas
Si tiene alguna duda sobre la implementación de está clase, puede contactarnos a: 

desarrollo@facturacionmoderna.com 

[1]: https://github.com/facturacionmoderna/FacturacionModerna-CSharp/blob/master/TimbradoCancelado/TimbradoCancelado/frmEjemplos.cs

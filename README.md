# JDK1.6支持TLS1.2协议,并忽略身份验证
TLS1.2升级成TLS1.1,项目中的区别,就访问时由HTTP变成了HTTPS
JDK1.6其实不支持TLS1.2,默认由TLS1.1访问请求.JDK1.8默认TLS1.2,比较简单的方案是升级JDK,但是对于老项目,升级JDK无非是<br>
挑灯上厕所.在不能升级JDK的情况下,可以使用一下方式
##不得不说的JDK和TLS的爱恨情仇
![Image text](https://github.com/Vico-cuiym/TLSSocketConnectionHttps.github.io/blob/master/com/img/JDK%E5%92%8CTLS%E7%9A%84%E7%88%B1%E6%81%A8%E6%83%85%E4%BB%87.png)
## 调用详细步骤
* 1、引入依赖 <br>
<pre><code>&lt;dependency&gt;
        &lt;groupId&gt;org.bouncycastle&lt;/groupId&gt;
        &lt;artifactId&gt;bcprov-jdk15on&lt;/artifactId&gt;
        &lt;version&gt;1.54&lt;/version&gt;
&lt;/dependency&gt;</code></pre>
* 2、创建协议工厂
```java
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.DataOutputStream;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.InetAddress;
import java.net.InetSocketAddress;
import java.net.Socket;
import java.net.UnknownHostException;
import java.security.KeyStore;
import java.security.Principal;
import java.security.SecureRandom;
import java.security.Security;
import java.security.cert.CertificateExpiredException;
import java.security.cert.CertificateFactory;
import java.util.Hashtable;
import java.util.LinkedList;
import java.util.List;

import javax.net.ssl.HandshakeCompletedListener;
import javax.net.ssl.SSLPeerUnverifiedException;
import javax.net.ssl.SSLSession;
import javax.net.ssl.SSLSessionContext;
import javax.net.ssl.SSLSocket;
import javax.net.ssl.SSLSocketFactory;
import javax.security.cert.X509Certificate;

import org.bouncycastle.crypto.tls.Certificate;
import org.bouncycastle.crypto.tls.CertificateRequest;
import org.bouncycastle.crypto.tls.DefaultTlsClient;
import org.bouncycastle.crypto.tls.ExtensionType;
import org.bouncycastle.crypto.tls.TlsAuthentication;
import org.bouncycastle.crypto.tls.TlsClientProtocol;
import org.bouncycastle.crypto.tls.TlsCredentials;
import org.bouncycastle.jce.provider.BouncyCastleProvider;

public class TLSSocketConnectionFactory extends SSLSocketFactory  {	
    static {
        if (Security.getProvider(BouncyCastleProvider.PROVIDER_NAME) == null) {
            Security.addProvider(new BouncyCastleProvider());
        }
    }
    @Override
    public Socket createSocket(Socket socket, final String host, int port,
    		boolean arg3) throws IOException {
        if (socket == null) {
            socket = new Socket();
        }
        if (!socket.isConnected()) {
            socket.connect(new InetSocketAddress(host, port));
        }
        final TlsClientProtocol tlsClientProtocol = new     TlsClientProtocol(socket.getInputStream(), socket.getOutputStream(), new     SecureRandom());
        return _createSSLSocket(host, tlsClientProtocol);
    }
    @Override public String[] getDefaultCipherSuites() { return null; }
    @Override public String[] getSupportedCipherSuites() { return null; }
    @Override public Socket createSocket(String host, int port) throws IOException, UnknownHostException { return null; }
    @Override public Socket createSocket(InetAddress host, int port) throws IOException { return null; }
    @Override public Socket createSocket(String host, int port, InetAddress localHost, int localPort) throws IOException, UnknownHostException { return null; }
    @Override public Socket createSocket(InetAddress address, int port, InetAddress localAddress, int localPort) throws IOException { return null; }
    private SSLSocket _createSSLSocket(final String host, final TlsClientProtocol tlsClientProtocol) {
        return new SSLSocket() {
            private java.security.cert.Certificate[] peertCerts;
            @Override public InputStream getInputStream() throws IOException { return tlsClientProtocol.getInputStream(); }
            @Override public OutputStream getOutputStream() throws IOException { return tlsClientProtocol.getOutputStream(); }
            @Override public synchronized void close() throws IOException { tlsClientProtocol.close(); }
            @Override public void addHandshakeCompletedListener( HandshakeCompletedListener arg0) { }
            @Override public boolean getEnableSessionCreation() { return false; }
            @Override public String[] getEnabledCipherSuites() { return null; }
            @Override public String[] getEnabledProtocols() { return null; }
            @Override public boolean getNeedClientAuth() { return false; }
            @Override
            public SSLSession getSession() {
                return new SSLSession() {
                    /*原本这些方法都是直接throw UnsupportedOperationException 导致看不到真实异常*/
                    @Override
                    public int getApplicationBufferSize() {
                        return 0;
                    }
                    @Override public String getCipherSuite() { return null; }
                    @Override public long getCreationTime() { return 0; }
                    @Override public byte[] getId() { return null; }
                    @Override public long getLastAccessedTime() { return 0; }
                    @Override public java.security.cert.Certificate[] getLocalCertificates() { return null; }
                    @Override public Principal getLocalPrincipal() { return null; }
                    @Override public int getPacketBufferSize() { return 0; }
                    @Override public X509Certificate[] getPeerCertificateChain() throws SSLPeerUnverifiedException { return null; }
                    @Override public java.security.cert.Certificate[] getPeerCertificates() throws SSLPeerUnverifiedException { return peertCerts; }
                    @Override public String getPeerHost() { return null; }
                    @Override public int getPeerPort() { return 0; }
                    @Override public Principal getPeerPrincipal() throws SSLPeerUnverifiedException { return null; }
                    @Override public String getProtocol() { return null; }
                    @Override public SSLSessionContext getSessionContext() { return null; }
                    @Override public Object getValue(String arg0) { return null; }
                    @Override public String[] getValueNames() { return null; }
                    @Override public void invalidate() { return; }
                    @Override public boolean isValid() { return true; }
                    @Override public void putValue(String arg0, Object arg1) { return; }
                    @Override
                    public void removeValue(String arg0) {
                        return;
                    }
                };
            }
            @Override public String[] getSupportedProtocols() { return null; }
            @Override public boolean getUseClientMode() { return false; }
            @Override public boolean getWantClientAuth() { return false; }
            @Override public void removeHandshakeCompletedListener(HandshakeCompletedListener arg0) { }
            @Override public void setEnableSessionCreation(boolean arg0) { }
            @Override public void setEnabledCipherSuites(String[] arg0) { }
            @Override public void setEnabledProtocols(String[] arg0) { }
            @Override public void setNeedClientAuth(boolean arg0) { }
            @Override public void setUseClientMode(boolean arg0) { }
            @Override public void setWantClientAuth(boolean arg0) { }
            @Override public String[] getSupportedCipherSuites() { return null; }
            @Override
            public void startHandshake() throws IOException {
                tlsClientProtocol.connect(new DefaultTlsClient() {
                    @SuppressWarnings("unchecked")
                    @Override
                    public Hashtable<Integer, byte[]> getClientExtensions() throws IOException {
                        Hashtable<Integer, byte[]> clientExtensions = super.getClientExtensions();
                        if (clientExtensions == null) {
                            clientExtensions = new Hashtable<Integer, byte[]>();
                        }
                        //Add host_name
                        byte[] host_name = host.getBytes();
                        final ByteArrayOutputStream baos = new ByteArrayOutputStream();
                        final DataOutputStream dos = new DataOutputStream(baos);
                        dos.writeShort(host_name.length + 3);
                        dos.writeByte(0);
                        dos.writeShort(host_name.length);
                        dos.write(host_name);
                        dos.close();
                        clientExtensions.put(ExtensionType.server_name, baos.toByteArray());
                        return clientExtensions;
                    }
                    @Override
                    public TlsAuthentication getAuthentication() throws IOException {
                        return new TlsAuthentication() {
                            @Override
                            public void notifyServerCertificate(Certificate serverCertificate) throws IOException {
                                try {
                                    KeyStore ks = _loadKeyStore();
                                    CertificateFactory cf = CertificateFactory.getInstance("X.509");
                                    List<java.security.cert.Certificate> certs = new LinkedList<java.security.cert.Certificate>();
                                    boolean trustedCertificate = false;
                                    for ( org.bouncycastle.asn1.x509.Certificate c : serverCertificate.getCertificateList()) {
                                        java.security.cert.Certificate cert = cf.generateCertificate(new ByteArrayInputStream(c.getEncoded()));
                                        certs.add(cert);
                                        String alias = ks.getCertificateAlias(cert);
                                        if(alias != null) {
                                            if (cert instanceof java.security.cert.X509Certificate) {
                                                try {
                                                    ( (java.security.cert.X509Certificate) cert).checkValidity();
                                                    trustedCertificate = true;
                                                } catch(CertificateExpiredException cee) {
                                                    // Accept all the certs!
                                                }
                                            }
                                        } else {
                                            // Accept all the certs!
                                        }
                                    }
                                    if (!trustedCertificate) {
                                        // Accept all the certs!
                                    }
                                    peertCerts = certs.toArray(new java.security.cert.Certificate[0]);
                                } catch (Exception ex) {
                                    ex.printStackTrace();
                                    throw new IOException(ex);
                                }
                            }
                            @Override
                            public TlsCredentials getClientCredentials(CertificateRequest certificateRequest) throws IOException {
                                return null;
                            }
                            private KeyStore _loadKeyStore() throws Exception {
                                FileInputStream trustStoreFis = null;
                                try {
                                    KeyStore localKeyStore = null;
                                    String trustStoreType = System.getProperty("javax.net.ssl.trustStoreType")!=null?System.getProperty("javax.net.ssl.trustStoreType"):KeyStore.getDefaultType();
                                    String trustStoreProvider = System.getProperty("javax.net.ssl.trustStoreProvider")!=null?System.getProperty("javax.net.ssl.trustStoreProvider"):"";
                                    if (trustStoreType.length() != 0) {
                                        if (trustStoreProvider.length() == 0) {
                                            localKeyStore = KeyStore.getInstance(trustStoreType);
                                        } else {
                                            localKeyStore = KeyStore.getInstance(trustStoreType, trustStoreProvider);
                                        }
                                        char[] keyStorePass = null;
                                        String str5 = System.getProperty("javax.net.ssl.trustStorePassword")!=null?System.getProperty("javax.net.ssl.trustStorePassword"):"";

                                        if (str5.length() != 0) {
                                            keyStorePass = str5.toCharArray();
                                        }
                                        localKeyStore.load(trustStoreFis, keyStorePass);
                                        if (keyStorePass != null) {
                                            for (int i = 0; i < keyStorePass.length; i++) {
                                                keyStorePass[i] = 0;
                                            }
                                        }
                                    }
                                    return localKeyStore;
                                } finally {
                                    if (trustStoreFis != null) {
                                        trustStoreFis.close();
                                    }
                                }
                            }
                        };
                    }

                });
            } // startHandshake
        };
    }
}
```
* 3、书写POST调用工具
```java
import org.apache.http.Consts;

import javax.net.ssl.HttpsURLConnection;
import java.io.*;
import java.net.URL;

public class HttpsUtils {
    /**
     * content-type类型为xml方式发送post请求
     *
     * @param urlPath
     * @param data
     * @param charSet
     * @return
     */
    public static String postXml(String urlPath, String data, String charSet) {
        String result = httpPostData(urlPath, data, charSet, null, "application/xml", "application/xml");
        return result;
    }

    private static String httpPostData(String urlPath, String data, String charSet, String[] header, String contentType, String accpect) {
        String result = null;
        URL url = null;
        HttpsURLConnection httpurlconnection = null;
        OutputStreamWriter out = null;
        BufferedReader reader = null;
        try {
            url = new URL(urlPath);
            httpurlconnection = (HttpsURLConnection) url.openConnection();
            httpurlconnection.setSSLSocketFactory(new TSLSocketConnectionFactory());
            httpurlconnection.setDoInput(true);
            httpurlconnection.setDoOutput(true);

            if (header != null) {
                for (int i = 0; i < header.length; i++) {
                    String[] content = header[i].split(":");
                    httpurlconnection.setRequestProperty(content[0], content[1]);
                }
            }

            httpurlconnection.setRequestMethod("POST");
            httpurlconnection.setRequestProperty("Content-Type", contentType);
            if (null != accpect) {
                httpurlconnection.setRequestProperty("Accpect", accpect);
            }

            httpurlconnection.connect();
            out = new OutputStreamWriter(httpurlconnection.getOutputStream(), charSet); // utf-8编码
            out.append(data);
            out.flush();
            out.close();

            int code = httpurlconnection.getResponseCode();

            if (code == 200) {
                // 读取响应
                int length = (int) httpurlconnection.getContentLength();// 获取长度
                InputStream is = httpurlconnection.getInputStream();
                reader = new BufferedReader(new InputStreamReader(is));
                String line = reader.readLine();
                StringBuilder builder = new StringBuilder();
                while (line != null) {
                    builder.append(line);
                    line = reader.readLine();
                }
                result = builder.toString();
            } else {
                // TODO
            }
        } catch (Exception e) {
            // TODO
        } finally {
            url = null;
            if (httpurlconnection != null) {
                httpurlconnection.disconnect();
            }
            try {
                if (out != null) {
                    out.close();
                }
                if (reader != null) {
                    reader.close();
                }
            } catch (IOException e) {
                // TODO
            }
        }
        return result;
    }
```
4、调用示例
```java
public static void main(String[] args) throws Exception {
        String dtdXml = "<?xml version=\"1.0\" encoding=\"utf-8\"?><!DOCTYPE cXML SYSTEM \"http://xml.cxml.org/schemas/cXML/1.2.014/cXML.dtd\"><cXML timestamp=\"2017-01-19T19:56:30\" payloadID=\"bac4b4a82e3342da919c7b427ee0fef2\"><Header><From><Credential domain=\"NetworkID\"><Identity>JDVEP4DIDI</Identity></Credential></From><To><Credential domain=\"NetworkID\"><Identity>Didipur</Identity></Credential></To><Sender><Credential domain=\"NetworkID\"><Identity>JDVEP4DIDI</Identity><SharedSecret>OGNmNGM3OGYtNWJhYi00ZTUwLTk0YTYtODAwZDVmYTU4NjMx</SharedSecret></Credential><UserAgent>JD VEP</UserAgent></Sender></Header><Message><PunchOutOrderMessage><BuyerCookie>3e3e68a280f45796cc24e59573e88ef7</BuyerCookie><PunchOutOrderMessageHeader operationAllowed=\"edit\"><Total><Money currency=\"CNY\">102.00</Money></Total><Shipping><Money currency=\"CNY\">0.00</Money><Description xml:lang=\"zh-CN\">运费</Description></Shipping></PunchOutOrderMessageHeader><ItemIn quantity=\"1\"><ItemID><SupplierPartID>102196</SupplierPartID><SupplierPartAuxiliaryID>46666778472</SupplierPartAuxiliaryID></ItemID><ItemDetail><UnitPrice><Money currency=\"CNY\">102.00</Money></UnitPrice><Description xml:lang=\"zh-CN\">维氏VICTORINOX瑞士军刀星座系列双鱼座0.6223.2PISC</Description><UnitOfMeasure>EA</UnitOfMeasure></ItemDetail></ItemIn><ItemIn quantity=\"1\"><ItemID><SupplierPartID>150706</SupplierPartID><SupplierPartAuxiliaryID>46666778472</SupplierPartAuxiliaryID></ItemID><ItemDetail><UnitPrice><Money currency=\"CNY\">0.00</Money></UnitPrice><Description xml:lang=\"zh-CN\">锐步Reebok女短袖T恤R537589 M码</Description><UnitOfMeasure>EA</UnitOfMeasure></ItemDetail></ItemIn></PunchOutOrderMessage></Message></cXML>";
        String url = "https://www.baidu.com";
        String result = "";
        result = httpPostData(url, dtdXml, Consts.UTF_8.name(), null, "application/xml", "application/xml");
        System.out.println(result);
    }
```

# 🚨以下为参考文档🚨
[TLS 1.0、TLS 1.1、TLS 1.2之间的区别](https://blog.csdn.net/mrpre/article/details/77978293)<br>
[SSL与TLS区别以及介绍](https://blog.csdn.net/adrian169/article/details/9164385)<br>
[Maven下载](http://maven.apache.org/download.cgi)<br>
[Maven中settings详解](https://www.cnblogs.com/hongmoshui/p/10762272.html)<br>
[Maven中settings配置阿里云](https://github.com/Vico-cuiym/TLSSocketConnectionHttps.github.io/blob/master/com/mvn/settings.xml)<br>





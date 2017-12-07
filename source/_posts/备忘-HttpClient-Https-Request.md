---
title: '[备忘]HttpClient Https Request'
date: 2017-12-07 16:08:20
tags:
categories:
- IT
- java
- other
---

httpClient 4.4

```java
package com.medica.framework.utils;

import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.io.UnsupportedEncodingException;
import java.net.URLDecoder;
import java.net.URLEncoder;
import java.security.KeyManagementException;
import java.security.KeyStoreException;
import java.security.NoSuchAlgorithmException;
import java.security.cert.CertificateException;
import java.security.cert.X509Certificate;
import java.util.ArrayList;
import java.util.Date;
import java.util.HashMap;
import java.util.Iterator;
import java.util.List;
import java.util.Map;
import java.util.Map.Entry;
import java.util.Set;
import java.util.UUID;

import javax.net.ssl.HostnameVerifier;
import javax.net.ssl.SSLContext;
import javax.net.ssl.SSLSession;
import javax.net.ssl.TrustManager;
import javax.net.ssl.X509TrustManager;

import org.apache.http.HttpEntity;
import org.apache.http.client.config.RequestConfig;
import org.apache.http.client.entity.UrlEncodedFormEntity;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.conn.ssl.SSLConnectionSocketFactory;
import org.apache.http.entity.ContentType;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClientBuilder;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.impl.conn.PoolingHttpClientConnectionManager;
import org.apache.http.message.BasicNameValuePair;
import org.apache.http.ssl.SSLContextBuilder;
import org.apache.http.ssl.TrustStrategy;
import org.apache.http.util.EntityUtils;
import org.apache.log4j.Logger;


public class HttpClientUtil2 {

	protected static Logger LOG=Logger.getLogger(HttpClientUtil.class);

	/**
	 * 定义超时配置
	 */
	public enum RequestConfigType {
		
		/**
		 * 设置10分钟超时
		 */
		TIMEOUT_600000(600000,600000,600000),
		/**
		 * 设置10s超时
		 */
		TIMEOUT_10000(10000, 10000, 10000),
		/**
		 * 设置2s超时
		 */
		TIMEOUT_2000(2000, 2000, 2000),
		/**
		 * 设置0.5秒
		 */
		TIMEOUT_500(500, 500, 500);
		private RequestConfig requestConfig;

		RequestConfigType(int socketTimeout, int connectTimeout,
				int requestTimeout) {
			this.requestConfig = RequestConfig.custom()
					.setSocketTimeout(socketTimeout)
					.setConnectTimeout(connectTimeout)
					.setConnectionRequestTimeout(requestTimeout).build();
		}
	}
	
	public static final String UTF8 = "UTF-8";
	public static final String GBK = "GBK";
	public static final String GB2312 = "GB2312";

	public static String httpsPost(String url,
			Map<String, String> headers, String jsonString,
			RequestConfigType requestConfigType) {
		

        HttpClientBuilder httpClientBuilder = HttpClientBuilder.create();  
          
        configureHttpClient(httpClientBuilder);  
		
		CloseableHttpClient httpClient =createSSLClientDefault();
		
		HttpPost post = new HttpPost(url);
		post.setConfig(requestConfigType.requestConfig);

		if (headers != null) {
			for (Entry<String, String> e : headers.entrySet()) {
				post.addHeader(e.getKey(), e.getValue());
			}
		}
		if (jsonString != null) {
			StringEntity entity = new StringEntity(jsonString,
					ContentType.APPLICATION_JSON);
			post.setEntity(entity);
		}
		String response = null;
		try {
			HttpEntity entity = httpClient.execute(post).getEntity();
			response = EntityUtils.toString(entity, UTF8);
		} catch (Exception e) {
			throw new RuntimeException("error post data to " + url, e);
		} finally {
			post.releaseConnection();
			if(httpClient!=null)
			{
				try {
					httpClient.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
		}
		return response;

	}
	  public static void configureHttpClient(HttpClientBuilder clientBuilder)   
	    {  
	        try  
	        {  
	        	 SSLContext ctx = SSLContext.getInstance("TLS");    
	             X509TrustManager tm = new X509TrustManager()  
	             {    
	                     public void checkClientTrusted(X509Certificate[] chain,  String authType) throws CertificateException   
	                     {  
	                           
	                     }    
	                     public void checkServerTrusted(X509Certificate[] chain,  String authType) throws CertificateException   
	                     {  
	                           
	                     }    
	                     public X509Certificate[] getAcceptedIssuers()   
	                     {    
	                         return null;    
	                     }    
	             };    
	             ctx.init(null, new TrustManager[]{tm}, null);    
	               
	             clientBuilder.setSslcontext(ctx);  
	              
	        }catch(Exception e)  
	        {  
	            e.printStackTrace();  
	        }  
	    }  
	  
	  
	  private static CloseableHttpClient createSSLClientDefault() {
	        try {
	            SSLContext sslContext = new SSLContextBuilder().loadTrustMaterial(null, new TrustStrategy() {
	                //信任所有
	                public boolean isTrusted(X509Certificate[] chain, String authType) throws CertificateException {
	                    return true;
	                }
	            }).build();
	            SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(sslContext, new HostnameVerifier() {
					
					@Override
					public boolean verify(String hostname, SSLSession session) {
						return true;
					}
				});
	            return HttpClients.custom().setSSLSocketFactory(sslsf).build();
	        } catch (KeyManagementException e) {
	            e.printStackTrace();
	        } catch (NoSuchAlgorithmException e) {
	            e.printStackTrace();
	        } catch (KeyStoreException e) {
	            e.printStackTrace();
	        }
	        return HttpClients.createDefault();
	    }
}

```

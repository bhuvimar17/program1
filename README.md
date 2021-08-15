package com.mkyong;

import java.io.BufferedReader;
import java.io.DataOutputStream;
import java.io.InputStreamReader;
import java.io.UnsupportedEncodingException;
import java.net.CookieHandler;
import java.net.CookieManager;
import java.net.URL;
import java.net.URLEncoder;
import java.util.ArrayList;
import java.util.List;
import javax.net.ssl.HttpsURLConnection;
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;


public class HttpUrlConnectionExample {

  private List<String> cookies;
  private HttpsURLConnection conn;

  private final String USER_AGENT = "Mozilla/5.0";

  public static void main(String[] args) throws Exception {

    String url = "https://accounts.google.com/ServiceLoginAuth";
    String gmail = "https://mail.google.com/mail/";

    HttpUrlConnectionExample http = new HttpUrlConnectionExample();

    // make sure cookies is turn on
    CookieHandler.setDefault(new CookieManager());

    // 1. Send a "GET" request, so that you can extract the form's data.
    String page = http.GetPageContent(url);
    String postParams = http.getFormParams(page, "username@gmail.com", "password");

    // 2. Construct above post's content and then send a POST request for
    // authentication
    http.sendPost(url, postParams);

    // 3. success then go to gmail.
    String result = http.GetPageContent(gmail);
    System.out.println(result);
  }

  private void sendPost(String url, String postParams) throws Exception {

    URL obj = new URL(url);
    conn = (HttpsURLConnection) obj.openConnection();

    // Acts like a browser
    conn.setUseCaches(false);
    conn.setRequestMethod("POST");
    conn.setRequestProperty("Host", "accounts.google.com");
    conn.setRequestProperty("User-Agent", USER_AGENT);
    conn.setRequestProperty("Accept",
        "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8");
    conn.setRequestProperty("Accept-Language", "en-US,en;q=0.5");
    for (String cookie : this.cookies) {
        conn.addRequestProperty("Cookie", cookie.split(";", 1)[0]);
    }
    conn.setRequestProperty("Connection", "keep-alive");
    conn.setRequestProperty("Referer", "https://accounts.google.com/ServiceLoginAuth");
    conn.setRequestProperty("Content-Type", "application/x-www-form-urlencoded");
    conn.setRequestProperty("Content-Length", Integer.toString(postParams.length()));

    conn.setDoOutput(true);
    conn.setDoInput(true);

    // Send post request
    DataOutputStream wr = new DataOutputStream(conn.getOutputStream());
    wr.writeBytes(postParams);
    wr.flush();
    wr.close();

    int responseCode = conn.getResponseCode();
    System.out.println("\nSending 'POST' request to URL : " + url);
    System.out.println("Post parameters : " + postParams);
    System.out.println("Response Code : " + responseCode);

    BufferedReader in = 
             new BufferedReader(new InputStreamReader(conn.getInputStream()));
    String inputLine;
    StringBuffer response = new StringBuffer();

    while ((inputLine = in.readLine()) != null) {
        response.append(inputLine);
    }
    in.close();
    // System.out.println(response.toString());

  }

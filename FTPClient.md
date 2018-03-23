# FTP 파일전송
> lib : cos.jar, commons-net.jar

1.fileList.jsp
> 폴더 파일리스트 불러오기
~~~
<%@ page language="java" contentType="text/html; charset=euc-kr" %>
<%@ page import="com.oreilly.servlet.MultipartRequest" %>
<%@ page import="org.apache.commons.net.ftp.*" %>

<%
	try {
	  FTPClient fClient = new FTPClient();

	  fClient.setControlEncoding("EUC-KR");
	  FTPClientConfig conf = new	FTPClientConfig(FTPClientConfig.SYST_NT);
	  conf.setServerLanguageCode("ko");
	  fClient.configure(conf);
	  fClient.connect("220.11.111.11");   //host ip
	  int reply = fClient.getReplyCode();

		if(!FTPReply.isPositiveCompletion(reply)){
			fClient.disconnect();
		}else{
			if(fClient.isConnected()){
				fClient.login("id", "pass");  // id, password
				fClient.enterLocalPassiveMode();						
				fClient.setFileType(FTP.BINARY_FILE_TYPE);        
				
				FTPFile[] ftpfiles = fClient.listFiles("/files"); 
				if (ftpfiles != null) { 
					for (int i = 0; i < ftpfiles.length; i++) { 
						FTPFile file = ftpfiles[i];
						//out.println(file.getName());
						out.println(file.toString() + "<br/>"); 
					}
				}
			}
			fClient.logout();
			fClient.disconnect();
		}
	
	} catch(Exception se) {
		System.out.println("접속 에러 :"+ se);
	}  
%>
~~~

2.fileUpload.jsp
> 다른 서버에 파일 전송
~~~
<%@ page language="java" contentType="text/html; charset=euc-kr" %>
<%@ page import="com.oreilly.servlet.MultipartRequest" %>
<%@ page import="org.apache.commons.net.ftp.*" %>
<%@ page import="java.io.*" %>

<%
	String filename = null;
	String targetName = "/files";   //수신 디렉토리

	int maxSize = 1073741824;
	String path = request.getRealPath("/upload/aaa"); //송신 디렉토리

	try {
		MultipartRequest multi=new MultipartRequest(request, path, maxSize, "EUC-KR");
		filename = multi.getFilesystemName("filename");

		FTPClient fClient = new FTPClient();
		fClient.setControlEncoding("EUC-KR");
		FTPClientConfig conf = new	FTPClientConfig(FTPClientConfig.SYST_NT);
		conf.setServerLanguageCode("ko");
		fClient.configure(conf);
		fClient.connect("220.11.111.11");
		int reply = fClient.getReplyCode();

		if(!FTPReply.isPositiveCompletion(reply)){
			fClient.disconnect();
		}else{
			
			if(fClient.isConnected()){
				try{
						fClient.login("id", "pass");
						fClient.enterLocalPassiveMode();
						fClient.changeWorkingDirectory("/files");
						fClient.setFileType(FTP.BINARY_FILE_TYPE);        

						//파일을 전송한다
						boolean flag = false;
						InputStream input = null;
						File local = null;
						
						local = new File(path, filename);
						input = new FileInputStream(local);

						targetName = targetName + "/"+ filename;					

					 	try {
							if(fClient.storeFile(targetName, input)) {
								flag = true;
							}
						}catch(IOException e){
							System.out.println("파일 전송에러" + e);
						}
 
				}catch(Exception se){
					System.out.println("업로드 에러 :"+ se);
				} 
			}
			fClient.logout();
			fClient.disconnect();
		}

	} catch(Exception se) {
		System.out.println("에러 :"+ se);
	}  
%>

~~~




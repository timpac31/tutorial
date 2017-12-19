# PhotoGallery Tutorial
> image preview, HTML5 multi select 구현
+ LIBS <br/>
 jai_core.jar, jai_imageid.jar, commons-fileupload-1.3.2.jar, commons-io-1.3.2.jar, commons-codec-1.3.jar, json-simple.jar
+ ISSUE <br/>
 파일 다중선택시 oreilly multipart upload library에서는 파일 하나밖에 못 가져와서 지원이 안됨 <br/>
 apache commons fileupload 로 구현
 
## write.jsp
~~~
<script>
function loadingShow() {
		$('#loadingWrap').show();
	}

	function loadingHide() {
		$('#loadingWrap').hide();
	}

	function uploadFile() {
	    var formfiles = $('#file')[0].files;
	    var len = formfiles.length;

    	var formData = new FormData();      
        for(var i=0; i<len; i++){
        	if(!formfiles[i].name.match(/\.(gif|jpg|jpeg|png)$/i)) {
        		alert('이미지 파일만 업로드 할 수 있습니다.');
        		return false;	    		
        	}
            formData.append('uploadFile', formfiles[i]);
        }
	 
	    $.ajax({
	        url: 'imageUploadProc.jsp',
	        data: formData,
	        processData: false,
	        contentType: false,
	        type: 'POST',
	        success: function (data) {
	            var jdata = JSON.parse(data);
	    		
	    		for(var i=0, ii=jdata.fileList.length; i<ii; i++) {
	    			$('#imagebox').append('<div id="'+ jdata.fileList[i].fileSeq +'">');
					  $('#imagebox').append('<img src="/upfiles/photowithmayor/' + jdata.fileList[i].thumbName + '" />');
            $('#imagebox').append('<input type="hidden" name="fileSeq" value="' + jdata.fileList[i].fileSeq + '" />');
	    			$('#imagebox').append('<a href="javascript:imageDel(' + jdata.fileList[i].fileSeq + ');">');
            $('#imagebox').append('<img src="/new_portal/ksnet/photo/btn_a_delete4.gif" alt="삭제" class="delete" /></a></div>');
	    		}
	    		$('#imagebox').css('background-image', 'none');
	        },
	        complete: function() {
	        	loadingHide();
	        }
	    });

	}
</script>

<table>
<tr>
  <th scope="row"><b>*</b><span class="hidden_item">필수입력</span>&nbsp;<label for="p_photo">사진첨부</label></th>
	<td>
    <div class="filebox"> 
	  <label for="file">사진 업로드 하기</label>
	  <input type="file" name="file" id="file" onchange="uploadFile();"  multiple />
		</div>
		<div id="imagebox"></div>
	</td>
</tr>  
<table>
~~~
## imageUploadProc.jsp
~~~
<%@ page language="java" contentType="text/html; charset=euc-kr" pageEncoding="euc-kr" %>
<%@ page import="java.awt.Graphics2D,java.awt.image.BufferedImage,java.io.File,java.io.IOException"%>
<%@ page import="javax.imageio.ImageIO,javax.media.jai.JAI,javax.media.jai.RenderedOp"%>
<%@ page contentType="text/html; charset=euc-kr" import="java.util.*,java.io.*,org.apache.commons.fileupload.*"%>
<%@ page import="org.json.simple.JSONArray, org.json.simple.JSONObject" %> 
<%!
	static public boolean createImage(String loadFile, String saveFile,int width,int height) throws IOException { 
		File  save = new File(saveFile);
		RenderedOp  rOp = JAI.create("fileload", loadFile);
		BufferedImage im = rOp.getAsBufferedImage();
		int imgWidth = width;
		int imgHeight = height;
		BufferedImage thumb = new BufferedImage(imgWidth, imgHeight, BufferedImage.TYPE_INT_RGB);
		Graphics2D  g2 = thumb.createGraphics();
		g2.drawImage(im, 0, 0, imgWidth, imgHeight, null);
		return ImageIO.write(thumb, "jpg", save);  
	}
%>

	
<%	
	/* 사진 파일, 썸네일 파일 업로드 start */
	if (!ServletFileUpload.isMultipartContent(request)) return;

	ArrayList fileList = new ArrayList();
	ArrayList fileSeqList = new ArrayList();
	
	int MEMORY_THRESHOLD = 1024 * 1024 * 3;
	int MAX_FILE_SIZE = 1024 * 1024 * 30;
	int MAX_REQUEST_SIZE = 1024 * 1024 * 50;

	DiskFileItemFactory factory = new DiskFileItemFactory();
	factory.setSizeThreshold(MEMORY_THRESHOLD);
	ServletContext servletContext = this.getServletConfig().getServletContext();
	File repository = (File) servletContext.getAttribute("javax.servlet.context.tempdir");
	factory.setRepository(repository);
	
	ServletFileUpload upload = new ServletFileUpload(factory);
	upload.setHeaderEncoding("UTF-8");
	upload.setFileSizeMax(MAX_FILE_SIZE);
	upload.setSizeMax(MAX_REQUEST_SIZE);
	
	String uploadPath = "D:\\file";
	File uploadDir = new File(uploadPath);
	
	if (!uploadDir.exists()) {
		uploadDir.mkdir();
	}

try {
	List formItems = upload.parseRequest(request);
	
	if (formItems != null && formItems.size() > 0) {
		for (int i=0; i<formItems.size(); i++) {
			FileItem item = (FileItem)formItems.get(i);
			if (!item.isFormField()) {
				String fileOriName = new File(item.getName()).getName();
				String fileNewName = System.currentTimeMillis() + (int)Math.random()*100000 + "." + fileOriName.substring(fileOriName.lastIndexOf(".") + 1);
				String filePath = uploadPath + File.separator + fileNewName;
				File storeFile = new File(filePath);
				
				item.write(storeFile);
				
				String thumbName = fileNewName.substring(0, fileNewName.lastIndexOf("."))+ "_145." + fileNewName.substring(fileNewName.lastIndexOf(".") + 1);
				createImage(filePath, uploadPath+File.separator+thumbName, 290, 190);
				
				HashMap fileMap = new HashMap();
				fileMap.put("fileOriName", fileOriName);
				fileMap.put("fileNmae", fileNewName);
				fileMap.put("thumbName", thumbName);
				fileList.add(fileMap);

			}
		}
	}
}catch (Exception ex) {
	System.err.println("파일업로드 에러 : " + ex);
}

	// List to JSON for return javascript
	JSONArray jarr = new JSONArray();
	JSONObject jObj = new JSONObject();
	JSONObject jSubObj = null;
	
	for(int j=0; j<fileList.size(); j++) {
		Map map = (HashMap)fileList.get(j);
		Set key = map.keySet(); 
		for (Iterator iterator = key.iterator(); iterator.hasNext();) { 
			String k = (String) iterator.next(); 
			String value = (String) map.get(k); 
			jSubObj = new JSONObject(); 
			jSubObj.put(k, value); 
		}
		jSubObj.put("fileSeq", fileSeqList.get(j));
		jarr.add(jSubObj); 
	}
	jObj.put("fileList", jarr);
	out.println(jObj);
	out.flush();
~~~

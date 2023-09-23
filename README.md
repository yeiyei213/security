talk t=getTalk("name1");
String senderdep = getParameter("senderdep");
String sender = getParameter("sender");
String sendermail = getParameter("sendermail");
String dep = getParameter("dep");
String name = getParameter("name");
String email = getParameter("email");
byte[] imageDataUrl = javax.xml.bind.DatatypeConverter.parseBase64Binary(getParameter("imageDataUrl"));
String base64Image = Base64.getEncoder().encodeToString(imageDataUrl);
String today=datetime.getToday("yy/mm/dd");
String time=datetime.getTime("h:m:s");
String date = today + " " + time;  

Vector paras = new Vector();
paras.addElement("[varchar]");
Vector res = t.callFromPool("sp_Get_cardID", paras);
String cardID = (String)(res.elementAt(0));

try{
	//新增ID+使用者輸入資料進資料庫
	String nowStr = getNow();
	String strSQL = "INSERT INTO Card (senderdep,sender,sendermail,dep,name,email,date,pic,id) values (?,?,?,?,?,?,?,?,?)";
	t.execFromPool(strSQL, new Object[]{senderdep,sender,sendermail,dep,name,email,date,imageDataUrl,cardID});

 
	//信件內容
	StringBuilder content = new StringBuilder();
	content.append("<h3><span style=\"font-size:18px\">2023感恩小卡來啦!!感謝有你 期待2024(´,,•ω•,,)♡<br/>")
		   .append("<img src='data:image/png;base64," + base64Image + "'/>");
     
	
	//寄信設定
	String host = "smtp-relay.gmail.com";   		
	String from = "dmsys_serv@children.org.tw";	 	
	String[] bcc = new String[]{email};   
	System.out.println(bcc+"***************");
	String subject = "2023感恩歡樂送ε٩(๑> ₃ <)۶з";
	String[] filename = null;
	String filepath = "";
	String content_type = "text/html; charset=utf-8";   

 	//看有沒有寄信成功 沒有就繼續 重複三次
	boolean success = false;
	int maxRetries = 3;
	int retryCount = 0;
	while (!success && retryCount < maxRetries) {									
		String sendRS = sendMailbccUTF8( host, from, bcc, subject, content.toString(), filename, filepath, content_type);
		if (sendRS.trim().equals("")) {
			message("上傳成功!已寄信通知");
			System.out.println("Sent mail notifications successfully....");
			success = true;
		} else {
			message("信件通知失敗，重寄中...");
			System.out.println("Sent mail notifications Error....");
			retryCount++;
			if (retryCount >= maxRetries) {
				message("寄信通知失敗，請自行提醒");
			} else {
				try {
					Thread.sleep(5000);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
	}
}catch(Exception exc1){}
return value;

talk t=getTalk("name1");
String sender = getParameter("sender");
String sendermail = getParameter("sendermail")+"@cwlf.org.tw";
String email = getParameter("email")+"@cwlf.org.tw";
String num = getParameter("num");
String imageUri= getParameter("imageUri");
String today=datetime.getToday("yy/mm/dd");
String time=datetime.getTime("h:m:s");
String date = today + " " + time;  
System.out.println("token:"+a);
Vector paras = new Vector();
paras.addElement("[varchar]");
Vector res = t.callFromPool("sp_Get_cardID", paras);
String cardID = (String)(res.elementAt(0));


String a = getParameter("a");
//有一隨機亂碼會在產生時存到資料庫且傳到前端html，當使用者打開網頁時，會從html將此亂碼回傳到後端，也就是參數a
//會拿a和資料庫內存儲的亂碼比對，如果有一樣就開始寄信流程，如果不一樣就不允許寄信
//然後因為寄信系統有時會秀逗所以有設定讓他最多執行三次，所以設了count想確定，如果count=1代表有寄信成功一次且不會再執行寄信
//人家是不是還得寫一個當他寄信成功後刷新亂碼的程式呀

String sl = "select sessionid from Sessionid where sessionid = ?";
String rt[][] = t.queryFromPool(sl,new Object[]{a});
System.out.println(rt[0][0]);
if (rt.length > 0 && rt[0][0].equals(a)) {
	int count = 0;
	if (count < 1) {
    	try {
			// 新增ID+使用者輸入資料進資料庫
			String nowStr = getNow();
			String strSQL = "INSERT INTO Card (sender,sendermail,email,date,id,base,get) values (?,?,?,?,?,?,?)";
			t.execFromPool(strSQL, new Object[]{sender, sendermail, email, date, cardID, imageUri, 1});
			System.out.println("11111111111111111111111111111111111");
			String sql = "select base from Card where id = ?";
			String ret[][] = t.queryFromPool(sql,new Object[]{cardID});
			String image = ret[0][0];
			int lastAmpersandIndex = image.lastIndexOf('&');
			String extractedString = image.substring(lastAmpersandIndex + 1);
			StringBuilder content = new StringBuilder();
			content.append("<!DOCTYPE html><html><body><h3><span style=\"font-size:18px\">2023感恩小卡來啦!!感謝有你 期待2024(\u00b4,,\u2022\u03c9\u2022,,)\u2661</span></h3>");	
			content.append("這是" + sender + "給你的感謝(\uff89>\u03c9<)\uff89<br/><br/><img src='"+extractedString+"'/></body></html>");
			
			//寄信設定
			String host = "smtp-relay.gmail.com";   		
			String from = "dmsys_serv@children.org.tw";	 	
			String[] bcc = new String[]{sendermail, email};
			String subject = "2023感恩歡樂送\u03b5\u0669(\u0e51> \u2083 <)\u06f6\u0437";
			String[] filename = new String[]{};
			String filepath = "";
			String content_type = "text/html; charset=utf-8";   
			boolean success = false;
			int maxRetries = 3;
			int retryCount = 0;
			while (retryCount < maxRetries) {									
				String sendRS = sendMailbccUTF8(host, from, bcc, subject, content.toString(), filename, filepath, content_type);
				if (sendRS.trim().equals("")) {
					message("上傳成功!已寄信通知");
					System.out.println("Sent mail notifications successfully....");
					count = 1;
					success = true;
				} else {
					message("信件通知失敗，重寄中...");
					System.out.println("Sent mail notifications Error....");
					retryCount++;
					count = 0;
					if (retryCount < maxRetries) {
						try {
							Thread.sleep(5000);
						} catch (InterruptedException e) {
							e.printStackTrace();
						}
					}
				}
			}
		}catch(Exception exc1){
			System.out.println("XX");
		}
	}else{
		System.out.println("XXXXX");
	}
}
return value;

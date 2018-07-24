---
title: JAVA PDF生成以及导出
date: 2018-07-24 11:02:22
tags: [java,PDF]
categories: 
  - java
  - PDF
---

之前做了个PDF生成以及导出参赛证功能，且里面带有二维码(二维码代码在其他文章)，觉得很棒，特此记录下

参赛证样板如图：![](11.png)

因为截图截的是正式数据，故打上马赛克，反正能看个大概模板样子就OK

废话不多说，直接上代码：

首先先导入jar包

```
	<dependency>
		<groupId>com.itextpdf</groupId>
		<artifactId>itextpdf</artifactId>
		<version>5.4.3</version>
	</dependency>
	<dependency>
		<groupId>com.itextpdf</groupId>
		<artifactId>itext-asian</artifactId>
		<version>5.2.0</version>
	</dependency>
```

PDFUtil代码如下：

```
import com.itextpdf.text.*;
import com.itextpdf.text.pdf.*;
import com.lowagie.text.Element;
import com.itextpdf.text.Font;
import com.itextpdf.text.Image;
import com.itextpdf.text.Rectangle;

import javax.servlet.http.HttpServletRequest;
import java.io.File;
import java.io.OutputStream;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class CreatePdfUtil implements PdfPCellEvent {
    /**
     * 导出准考证pdf（显示照片显示二维码）
     * @param list
     * @param out
     * @param request
     * @return
     */
    public static String exportExamCardPdf1(List list, OutputStream out, HttpServletRequest request) {
    	//设置PDF大小
        Rectangle pageSize = new Rectangle(291.0F, 429.0F);

        Document document = new Document(pageSize,0,0,0,0);
        try {
            //因为该功能有字体限制，所以用的微软雅黑字体，在项目根目录
            String wrHei= request.getSession().getServletContext().getRealPath("") + File.separator + "temp" + File.separator+"msyh.ttf" ;
            //设置PDF背景图，之前有想法弄个背景图，后来老大说不用就注释了
//            Image jpeg = Image.getInstance(request.getSession().getServletContext().getRealPath("") + File.separator + "/export/123.jpg");
//            jpeg.setAlignment(Image.UNDERLYING);
//            jpeg.setAbsolutePosition(0, 0);

            BaseFont bfHei = BaseFont.createFont(wrHei, BaseFont.IDENTITY_H, BaseFont.NOT_EMBEDDED);
            //设置中文字体
            BaseFont bfChinese = BaseFont.createFont("STSong-Light", "UniGB-UCS2-H", BaseFont.NOT_EMBEDDED);
            //设置不同大小字体以及加粗
            Font fontChinese = new Font(bfHei, 10, Font.NORMAL);
            Font fontChinese_1 = new Font(bfHei, 10, Font.BOLD);
            Font title1Chinese = new Font(bfChinese, 12, Font.BOLD);
            Font titleChinese = new Font(bfChinese, 22, Font.BOLD);
            //字体加上颜色 ，目前采用淡蓝色
            BaseColor color = new BaseColor(0,32,96);
            fontChinese_1.setColor(color);
            PdfWriter.getInstance(document, out);
            // 步骤 3:打开文档
            document.open();
//            document.add(jpeg);
            //创建一个表格
            PdfPTable outTable = new PdfPTable(1);
            //设置分页
            outTable.setSplitLate(false);
            String url = request.getSession().getServletContext().getRealPath("") ;
            String path1 = request.getContextPath();
            String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path1+"/";
            //把单元加到表格中
            if(list != null && list.size()>0){
                for(int i=0;i<list.size();i++){
                    //定义一个表格单元
                    Map<String, Object> map = (HashMap<String, Object>)list.get(i);
                    //获取标题
                    String title = map.get("title")==null?"":map.get("title").toString();
                    //创建带有三列的表格
                    PdfPTable table = new PdfPTable(3);
                    //设置表格内行边框消失
                    table.getDefaultCell().setBorder(0);
                    //设置表格内每列宽
                    table.setWidths(new float[] { 5f,5f, 8f});
                    //创建头部空白行
                    PdfPCell kbCell = new PdfPCell(new Paragraph("", titleChinese));
                    kbCell.setBorder(0);
                    kbCell.setFixedHeight(15.0f);
                    kbCell.setPaddingTop(20.0f);
                    kbCell.setPaddingBottom(2.f);
                    outTable.addCell(kbCell);
                   //创建头部空白行
                    PdfPCell cell0_1 =  new PdfPCell(new Paragraph(" ", title1Chinese));
                    cell0_1.setPaddingTop(8.0f);
                    cell0_1.setPaddingBottom(6.f);
                    cell0_1.setBorder(0);
                    outTable.addCell(cell0_1);
                    //创建标题行
                    PdfPCell cell0 =  new PdfPCell(new Paragraph(title, title1Chinese));
                    cell0.setPaddingTop(6.0f);
                    cell0.setPaddingBottom(4.0f);
                    cell0.setHorizontalAlignment(Element.ALIGN_CENTER);
                    cell0.setBorder(0);
                    cell0.disableBorderSide(2);
                    cell0.setColspan(3);
                    cell0.setMinimumHeight(20);
                    table.addCell(cell0);
                    PdfPCell cell1 =  new PdfPCell(new Paragraph("参赛证", titleChinese));
                    cell1.setPaddingTop(3.0f);
                    cell1.setPaddingBottom(18.0f);
                    cell1.setMinimumHeight(40);
                    cell1.setHorizontalAlignment(Element.ALIGN_CENTER);
                    cell1.setBorder(0);
                    cell0.disableBorderSide(2);
                    cell1.setColspan(3);
                    table.addCell(cell1);
                    //姓名
                    PdfPCell cell2 =  new PdfPCell(new Paragraph("  姓       名:", fontChinese_1));
                    cell2.setBorder(0);
                    cell2.setMinimumHeight(30);
                    table.addCell(cell2);
                    PdfPCell cell2_1 =  new PdfPCell(new Paragraph(StringUtil.getString(map.get("name")), fontChinese));
                    cell2_1.setBorder(0);
                    cell2_1.setMinimumHeight(30);
                    table.addCell(cell2_1);
                    //头像图片
                    String path12 = request.getSession().getServletContext().getRealPath("") + File.separator ;
                    String img = StringUtil.getString(map.get("imgPath"));
                    img=path12+img;
                    String imgUrl = "";
                    //判断图片路径是否为空。为空则设置默认图片
                    if("".equals(img)){
                        imgUrl = basePath+"/static/admin/images/mr3.jpg";
                    }else{
                        imgUrl=img;
                    }
                    Image image = null;
                    try {
                        image = Image.getInstance(imgUrl);
                    } catch (Exception e) {
                        e.printStackTrace();
                        //报错也设置为默认图片
                        imgUrl = basePath+"/static/admin/images/mr3.jpg";
                    }
                    image = Image.getInstance(imgUrl);
                    image.scaleAbsolute(80,110);
                    PdfPTable imgTable = new PdfPTable(1);
                    PdfPCell cell_img=new PdfPCell(image);
                    cell_img.setRowspan(4);
                    cell_img.setBorder(0);
                    cell_img.setPadding(0);
                    cell_img.setPaddingLeft(0.1f);
                    imgTable.addCell(cell_img);
                    PdfPCell cellImg =  new PdfPCell();
                    cellImg.addElement(imgTable);
                    cellImg.setRowspan(4);
                    cellImg.setBorder(0);
                    cellImg.setMinimumHeight(20);
                    table.addCell(cellImg);
                    //性别
                    PdfPCell cell3 =  new PdfPCell(new Paragraph("  性       别:", fontChinese_1));
                    cell3.setBorder(0);
                    cell3.setMinimumHeight(30);
                    table.addCell(cell3);
                    PdfPCell cell3_1 =  new PdfPCell(new Paragraph(StringUtil.getString(map.get("sex")), fontChinese));
                    cell3_1.setBorder(0);
                    cell3_1.setMinimumHeight(30);
                    table.addCell(cell3_1);
                    //入学时间
                    PdfPCell cell4 =  new PdfPCell(new Paragraph("  入学时间:", fontChinese_1));
                    cell4.setBorder(0);
                    cell4.setMinimumHeight(30);
                    table.addCell(cell4);
                    PdfPCell cell4_1 =  new PdfPCell(new Paragraph(StringUtil.getString(map.get("enrolmentTime")), fontChinese));
                    cell4_1.setBorder(0);
                    cell4_1.setMinimumHeight(30);
                    table.addCell(cell4_1);
                    //项目
                    PdfPCell cell5 =  new PdfPCell(new Paragraph("  项       目:", fontChinese_1));
                    cell5.setBorder(0);
                    cell5.setMinimumHeight(30);
                    table.addCell(cell5);
                    PdfPCell cell5_1 =  new PdfPCell(new Paragraph(StringUtil.getString(map.get("project")), fontChinese));
                    cell5_1.setBorder(0);
                    cell5_1.setMinimumHeight(30);
                    table.addCell(cell5_1);
                    //参赛单位
                    PdfPCell cell6 =  new PdfPCell(new Paragraph("  参赛单位:", fontChinese_1));
                    cell6.setBorder(0);
                    cell6.setMinimumHeight(20);
                    table.addCell(cell6);
                    PdfPCell cell6_1 =  new PdfPCell(new Paragraph(StringUtil.getString(map.get("competitionSchool")), fontChinese));
                    cell6_1.setBorder(0);
                    cell6_1.setColspan(2);
                    cell6_1.setMinimumHeight(30);
                    table.addCell(cell6_1);
                    //注册号码
                    PdfPCell cell7 =  new PdfPCell(new Paragraph("  注册编号:", fontChinese_1));
                    cell7.setBorder(0);
                    cell7.setMinimumHeight(30);
                    table.addCell(cell7);
                    PdfPCell cell7_1 =  new PdfPCell(new Paragraph(StringUtil.getString(map.get("registrationNumber")), fontChinese));
                    cell7_1.setBorder(0);
                    cell7_1.setColspan(2);
                    table.addCell(cell7_1);
                    //固定标识
                    PdfPCell c2=new PdfPCell(new Paragraph("  XXX学生体育协会", fontChinese));
                    c2.setBorder(0);
                    c2.setColspan(2);
                    c2.setPaddingBottom(15.0f);
                    c2.setVerticalAlignment(Element.ALIGN_BOTTOM);
                    table.addCell(c2);
                    //二维码
                    String text =StringUtil.getString(map.get("url")); // 获取需要生成二维码的路径
                    String temp = java.util.UUID.randomUUID().toString();
                    int width = 100; // 二维码图片的宽
                    int height = 100; // 二维码图片的高
                    String format = "png"; // 二维码图片的格式
                    //指定二维码生成存放的路径
                    String path = request.getSession().getServletContext().getRealPath("") + File.separator + "export" + File.separator + "erweima" + File.separator + "temp";
                    File file = new File(path);
                    if(!file.exists()){
                        file.mkdirs();
                    }
                    String fileName=temp + ".png";
                    File file1 = new File(file,fileName);
                    try {
                        if(!file1.exists()) {
                            file1.createNewFile();
                        }
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                    // 生成二维码图片，并返回图片路径
                    String QR_path = QRCodeUtil.generateQRCode(text, width, height, format, file1.getPath());
                    Image imageQr1 = null;
                    try {
                        imageQr1 = Image.getInstance(QR_path);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                    PdfPTable imgTableQr1 = new PdfPTable(1);
                    imageQr1.scaleAbsolute(90,90);
                    PdfPCell cell_imgQr1=new PdfPCell(imageQr1);
                    cell_imgQr1.setBorder(0);
                    cell_imgQr1.setPadding(0);
                    imgTableQr1.addCell(cell_imgQr1);
                    PdfPCell cellImgQr2 =  new PdfPCell(cell_imgQr1);
                    cellImgQr2.addElement(imgTableQr1);
                    cellImgQr2.setBorder(0);
                    cellImgQr2.setPadding(0);
                    cellImgQr2.setPaddingLeft(0.0f);
                    cellImgQr2.setPaddingBottom(5.0f);
                    table.addCell(cellImgQr2);
                    outTable.addCell(table);
                    //设置底部空白行，以至于控制table出于中心位置，且不会因为数量而导致被换行等样式错乱
                    PdfPCell cell0_1_1 =  new PdfPCell(new Paragraph(" ", title1Chinese));
                    cell0_1_1.setPaddingTop(20.0f);
                    cell0_1_1.setPaddingBottom(6.f);
                    cell0_1_1.setBorder(0);
                    outTable.addCell(cell0_1_1);
                }
            }
            //增加到文档中
            document.add(outTable);
            // 步骤 5:关闭文档
            document.close();
            return "true";
        } catch (Exception de) {
            System.err.println(de.getMessage());
            return "false";
        }
    }


    @Override
    public void cellLayout(PdfPCell pdfPCell, Rectangle position, PdfContentByte[] pdfContentBytes) {
        PdfContentByte cb = pdfContentBytes[PdfPTable.LINECANVAS];
        cb.saveState();
        //cb.setLineCap(PdfContentByte.LINE_CAP_ROUND);
        //cb.setLineDash(0, 1, 1);
        cb.setLineWidth(0.5f);
        cb.setLineDash(new float[] { 5.0f, 5.0f }, 0);
        cb.moveTo(position.getLeft(), position.getBottom());
        cb.lineTo(position.getRight(), position.getBottom());
        cb.lineTo(position.getRight(), position.getTop());
        cb.lineTo(position.getLeft(), position.getTop());
        cb.lineTo(position.getLeft(), position.getBottom());
        cb.stroke();
        cb.restoreState();
    }
}

```

至于怎么调用嘛，也简单

```
 String path = request.getSession().getServletContext().getRealPath("") + File.separator + "export" + File.separator + "pdf" + File.separator + "temp";
        String temp = java.util.UUID.randomUUID().toString();
        OutputStream out = null;
		File file = new File(path + File.separator );
		if(!file.exists()){
			file.mkdirs();
		}
		String fileName=temp + ".pdf";
		File file1 = new File(file,fileName);
			try {
				if(!file1.exists()) {
					file1.createNewFile();
				}
				out = new FileOutputStream(file1);
		} catch (Exception e) {
			e.printStackTrace();
		}

		schoolName = schoolName + "参赛证"+".pdf";
		HttpHeaders headers = new HttpHeaders();
		if(list.size()>0){
			CreatePdfUtil.exportExamCardPdf(list,out, request);
		}
        headers.setContentType(MediaType.APPLICATION_OCTET_STREAM);
        String downloadFileName = new String(schoolName.getBytes("UTF-8"), "ISO-8859-1");
        headers.setContentDispositionFormData("attachment", downloadFileName);
        return new ResponseEntity<byte[]>(FileUtils.readFileToByteArray(file1), headers, HttpStatus.OK);
```

至此全部代码就都结束了，有什么问题希望各位大佬给我指出，以至于帮助我更好的学习，感谢


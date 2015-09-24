---
layout: post
title:  "ReportLab"
date:   2015-09-24 14:26:05
categories: Python
---

* content
{:toc}

## 简介

ReportLab[官网](http://www.reportlab.com/)的强大之处在于能满足绝大部分报表的需求形式，可以在点这里[下载](https://pypi.python.org/pypi/reportlab/#downloads)。

* 要求 Python 版本为 2.7 or >=3.3
* 直接使用 API 是最麻烦、最基础的做法
* 支持 [RML](http://www.reportlab.com/software/rml-reference/) 和 HTML

## 安装
### pip 安装

	sudo pip install reportlab

### 源码安装

下载

	wget https://pypi.python.org/packages/source/r/reportlab/reportlab-3.2.0.tar.gz#md5=79d059e797c557aed4b40c68dd6c7eae

解压

	tar zxvf reportlab-3.2.0.tar.gz
	cd reportlab-3.2.0

安装

	#Ubuntu
	sudo python setup.py install

	#CentOS
	su root
	python setup.py install

测试

	python
	import reportlab

## 使用

### canvas

画布用法可从 API 的 Chapter 2 学习，示例代码如下：

	#!/usr/bin/python
	# -*- coding: utf-8 -*-
	# python version 2.7.6

	from reportlab.pdfgen import canvas 
	from reportlab.lib import fonts,colors
	from reportlab.lib.units import inch
	from reportlab.platypus import SimpleDocTemplate, Table, TableStyle
	from reportlab.lib.pagesizes import A4, cm
	from reportlab.lib.styles import getSampleStyleSheet
	from reportlab.lib.enums import TA_JUSTIFY, TA_LEFT, TA_CENTER
	from reportlab.platypus import Paragraph, Table, TableStyle
	import Image

	def hello(): 
		#指定pdf目录和文件名 
		c = canvas.Canvas("/home/lin/World.pdf", pagesize=A4)  

		#Logo和标题
		im = Image.open('/home/lin/work/png/portrait.png')
		c.drawInlineImage( im, 230,800, width=19,height=22)
		c.drawString(255,807,"NIPT") 

		c.drawString(75,707,"There is a table") 

		width, height = A4
		styles = getSampleStyleSheet()
		styleN = styles["BodyText"]
		styleN.alignment = TA_LEFT
		styleBH = styles["Normal"]
		styleBH.alignment = TA_CENTER


		# Headers
		hid = Paragraph('''<b>ID</b>''', styleBH)
		hname = Paragraph('''<b>Name</b>''', styleBH)
		hprice = Paragraph('''<b>Price</b>''', styleBH)

		# Texts
		tid = Paragraph('1', styleN)
		tname = Paragraph('cherry', styleN)
		tprice = Paragraph('$52.00', styleN)

		data= [[hid, hname, hprice],[tid, tname, tprice]]

		table = Table(data, colWidths=[5 * cm, 5 * cm, 5 * cm])

		table.setStyle(TableStyle([
			               ('INNERGRID', (0,0), (-1,-1), 0.25, colors.black),
			               ('BOX', (0,0), (-1,-1), 0.25, colors.black),
			               ]))

		table.wrapOn(c, width, height)
		table.drawOn(c, 75, 650)

		# 画圆 canvas.circle(x_cen, y_cen, r, stroke=1, fill=0)
		c.circle(100, 100, 100, stroke=1, fill=0)
		# 画线 canvas.line(x1,y1,x2,y2)
		c.line(100,200,30,50)

		c.showPage() 
		c.save() 

	if __name__=='__main__':
		hello()

### SimpleDocTemplate
画布用法可从 API 的 5.6 Documents and Templates 学习，示例代码如下：

	#!/usr/bin/python
	# -*- coding: utf-8 -*-
	# python version 2.7.6

	__des__ = 'NIPT create pdf'
	__author__ = 'lin'

	import os
	from reportlab.lib import colors
	from reportlab.lib.enums import TA_LEFT
	from reportlab.lib.pagesizes import A4,cm
	from reportlab.lib.styles import getSampleStyleSheet
	from reportlab.pdfbase import pdfmetrics
	from reportlab.pdfbase.ttfonts import TTFont
	from reportlab.platypus import Image , Paragraph , SimpleDocTemplate , Spacer , Table , TableStyle
	from tableutil import *

	pdfmetrics.registerFont(TTFont('hei', 'ttc/simhei.ttf'))

	# create pdf
	def createPDF(path):
		if(not path.endswith(os.sep)):
			path = path+os.sep
		datakey = str(os.path.split(path[:-1])[1])
		rdir = os.path.join(path,"Result")
		doc = SimpleDocTemplate(os.path.join(path,datakey+".pdf"),
			pagesize=A4,rightMargin=10,
			leftMargin=10,topMargin=10,bottomMargin=10)
		# define style
		styles = getSampleStyleSheet()
		# title style
		styleTitle = styles["Normal"]
		styleTitle.alignment = TA_LEFT
		styleTitle.leftIndent = 55
		# context style
		styleContext = styles["BodyText"]
		styleContext.alignment = TA_LEFT

		# total context
		tatal = []

		# logo and pdf title
		im = Image("/home/lin/work/png/portrait.png", 15, 17)
		pdftitle = '<font size=14 name="hei">NIPT 报告</font>'
		data= [[im, Paragraph(pdftitle, styleContext)]]
		table = Table(data, colWidths=[0.9 * cm, 3.5 * cm])
		table.setStyle(TableStyle([('VALIGN',(-1,-1),(-1,-1),'TOP')]))
		tatal.append(table)
		tatal.append(Spacer(1, 12))

		# first table
		t1 = '<font size=10 name="hei">数据统计：</font>'
		tatal.append(Paragraph(t1, styleTitle))
		tatal.append(Spacer(1, 12))
		firstData = simpleTable(os.path.join(rdir,"Data_report.txt"),styleContext,styleContext)
		table = Table(firstData,colWidths=[4.0 * cm, 4.0 * cm, 2.0 * cm, 4.0 * cm, 2.0 * cm])
		table.setStyle(TableStyle([
		               ('INNERGRID', (0,0), (-1,-1), 0.25, colors.black),
		               ('BOX', (0,0), (-1,-1), 0.25, colors.black),
		               ]))
		tatal.append(table)
		tatal.append(Spacer(1, 12))

		# second table
		t2 = '<font size=10 name="hei">报告：</font>'
		tatal.append(Paragraph(t2, styleTitle))
		tatal.append(Spacer(1, 12))
		secondData = simpleTable(os.path.join(rdir,"Zscore.txt"),styleContext,styleContext)
		table = Table(secondData,colWidths=[4.0 * cm, 4.0 * cm, 4 * cm, 4* cm])
		table.setStyle(TableStyle([
		               ('INNERGRID', (0,0), (-1,-1), 0.25, colors.black),
		               ('BOX', (0,0), (-1,-1), 0.25, colors.black),
		               ]))
		tatal.append(table)
		tatal.append(Spacer(1, 12))

		# Pic
		ptext = '<font size=10 name="hei">染色体：</font>'
		tatal.append(Paragraph(ptext, styleTitle))
		tatal.append(Spacer(1, 12))
		im = Image(os.path.join(rdir,"Pic.txt.mini.png"), 465, 108)
		tatal.append(im)

		doc.build(tatal)

	if __name__=='__main__':
		createPDF("/home/lin/work/20141211603127")

### 中文支持

ReportLab 本身不支持中文，要在 PDF 中显示中文，需要手动引入中文字体包。
在 C:\WINDOWS\Fonts 下寻找需要的字体，将其引入 Python 程序，利用 pdfmetrics 方法注册字体后即可使用。

	#!/usr/bin/python
	# -*- coding: utf-8 -*-
	# python version 2.7.6

	from reportlab.lib.styles import getSampleStyleSheet
	from reportlab.platypus import SimpleDocTemplate,Paragraph
	from reportlab.pdfbase import pdfmetrics
	from reportlab.pdfbase.ttfonts import TTFont

	pdfmetrics.registerFont(TTFont('hei', 'ttc/simsun.ttc'))
	 
	stylesheet=getSampleStyleSheet()
	elements = []
	doc = SimpleDocTemplate('/home/lin/test.pdf')
	elements.append(Paragraph('<font name="hei">学 生 成 绩 单</font>',stylesheet['Title']))
	doc.build(elements)

（未完待续）

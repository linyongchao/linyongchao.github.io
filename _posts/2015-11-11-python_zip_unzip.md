---
layout: post
title:  Python 压缩及解压缩文件
date:   2015-11-12 01:19:05
categories: Python
---

* content
{:toc}

	import os
	import zipfile
	 
	def zip_dir(dirname,zipfilename):
	    filelist = []
	    if os.path.isfile(dirname):
	        filelist.append(dirname)
	    else :
	        for root, dirs, files in os.walk(dirname):
	            for name in files:
	                filelist.append(os.path.join(root, name))
	         
	    zf = zipfile.ZipFile(zipfilename, "w", zipfile.zlib.DEFLATED)
	    for tar in filelist:
	        arcname = tar[len(dirname):]
	        zf.write(tar,arcname)
	    zf.close()
	 
	 
	def unzip_file(zipfilename, unziptodir):
	    if not os.path.exists(unziptodir): os.mkdir(unziptodir, 0777)
	    zfobj = zipfile.ZipFile(zipfilename)
	    for name in zfobj.namelist():
	        name = name.replace('\\','/')
	       
	        if name.endswith('/'):
	            os.mkdir(os.path.join(unziptodir, name))
	        else:            
	            ext_filename = os.path.join(unziptodir, name)
	            ext_dir= os.path.dirname(ext_filename)
	            if not os.path.exists(ext_dir) : os.mkdir(ext_dir,0777)
	            outfile = open(ext_filename, 'wb')
	            outfile.write(zfobj.read(name))
	            outfile.close()
	 
	if __name__ == '__main__':
	    zip_dir('utils','u.zip')
	    unzip_file('u.zip','t')


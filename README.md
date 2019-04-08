Simple Webscan Python script


0x00:
这是一个基本上最简单的Python目录扫描脚本；用的是御剑的字典，用来练习的简单脚本。所以需要自己找御剑的字典。


0x01:源代码


import requests
import queue
import sys
import os
import threading
from optparse import OptionParser
from time import time


headers = {
	'User-Agent':'',
	'Referer': 'https://www.baidu.com/'	,
}

class Dscanner(object):

	def __init__(self,target,ext,thread_count):
		self.target = target
		self.file_ext = ext
		self.thread_count = thread_count
		self.queue = queue.Queue()
		self.total = ""
		self.result = []
		self.W = '\033[0m'
		self.G = '\033[1;32m'
		self.O = '\033[1;33m'
		self.time = time()

	def run(self,queue):
		while not self.queue.empty():
			self.print_msg()
			url = self.queue.get()
			try:
				r = requests.get(url=url, headers=headers,timeout= 10,)
				if r.status_code == 200:
					self.result.append(url)
					sys.stdout.write(self.G + '\r'+'[0K] %s\r\n'%url + self.W)
			except Exception as e:
				print(e)
				#pass

	def print_msg(self):
		msg = "剩余{}次|总共{}次".format(self.queue.qsize(),self.total)
		sys.stdout.write('\r'+"[*]"+msg)

	def start(self):

		print('-'*60)
		print(u'{}[-] 正在扫描地址: {}\t当前线程数：{}{}'.format(self.O,self.target, self.thread_count,self.W))
		if self.file_ext == 1:
			print(u'{}[-] 当前扫描类型: All file types !!!{}'.format(self.O,self.W))
		else:
			print(u'{}[-] 当前扫描类型: {}{}'.format(self.O,self.file_ext,self.W))
		print('-'*60)

		try:
			if self.file_ext == 1:
				self.file_ext = os.listdir("dicts")
				for name in self.file_ext:
					ext = name.split('.')[0]
					with open('./dicts/%s.txt' % ext,'r') as f:
						for line in f.readlines():
							self.queue.put(self.target+line.rstrip('\n'))
			elif self.file_ext:
				file_name = ['dir','mdb']
				file_name.append(self.file_ext)
				for name in file_name:
					with open('./dicts/%s.txt' % name,'r') as f:
						for line in f.readlines():
							self.queue.put(self.target+line.rstrip('\n'))

			self.total = self.queue.qsize()

			threads = []

			for i in range(0,int(self.thread_count)+1):
				t = threading.Thread(target=self.run,args=(self.queue,))
				t = threads.append(t)
				
			for t in threads:
				t.setDaemon(True)
				t.start()
				queue.task_done()

			self.queue.join()

			print('-'*60)
			print(u'{}[-] 扫描完成耗时: {} 秒.{}'.format(self.O,time()-self.time, self.W))
		except Exception as e:
			print(e)
		except KeyboardInterrupt:
			print(self.R + u'\n[-] 用户终止扫描...' + self.W)
			sys.exit(1)

if __name__ == '__main__':
	logo=''' 
		welcome to WEBscan
	'''
	print('\033[1;34m'+ logo +'\033[0m')

	usage = '''
 
If no file type is specified, default full scan.
If the number of threads is not specified, the default is 10.

Usage:python dirscan.py -u http://zone.secevery.com
      python dirscan.py -u http://zone.secevery.com -f php 
      python dirscan.py -u http://zone.secevery.com -f php -n 100
			'''


	parser =OptionParser()

	parser.add_option("-u" ,"--url", action="store",type="string",dest="url",help="Please input target url")
	parser.add_option("-f", "--file",action="store",type="string",dest="file",help="Please input test target file_ext")
	parser.add_option("-n", "--num",action="store",type="int",dest="count",help="thread counts for scan,default 10.")

	(options,args) = parser.parse_args()

	if options.url and options.file and options.count:
		scan = Dscanner(options.url,options.file,options.count)
		scan.start()
	elif options.url and options.file and options.count == None:	
		thread_count = 10
		scan = Dscanner(options.url,options.file,thread_count)
		scan.start()
	elif options.url and options.file == None and options.count == None:	
		flag = 1
		thread_count = 10
		scan = Dscanner(options.url,flag,thread_count)
		scan.start()
	else:
		print('\033[1;32m'+ usage +'\033[0m')
		parser.print_help()

# wx

from serial import Serial
from serial.tools import list_ports
import os
import xively
import subprocess
import time
import datetime
import requests
import csv

baud = 9600

FEED_ID = "916945674"
API_KEY = "HDnSlXQneT85SAeWmudL11VlDwyuYoVV5mFHH8grHYKsWy9C"

# initialize api client
api = xively.XivelyAPIClient(API_KEY)

#Scan USB ports, and find the serial connection to the Arduino
port_list = list_ports.comports()
for line in port_list:
	if('usb' in line[2].lower()):
		chosenPort = line[0]
ser = Serial('/dev/ttyACM0',baud)

dt=str(datetime.datetime.now().strftime("DHT%y-%m-%d_%H%M"))

myfile = open(dt + ".txt", "w")
filename = str(dt + ".txt")

# function to return a datastream object. This either creates a new datastream,
# or returns an existing one
def get_datastreamTemp(feed):
  try:
    datastreamDHT1temp = feed.datastreams.get("DHT1_Temperature")
    return datastreamDHT1temp
  except:
    datastreamDHT1temp = feed.datastreams.create("DHT1_Temperature", tags="temperature")
    return datastreamDHT1temp

def get_datastreamTemp2(feed):
  try:
    datastreamDHT2temp = feed.datastreams.get("DHT2_Temperature")
    return datastreamDHT2temp
  except:
    datastreamDHT2temp = feed.datastreams.create("DHT2_Temperature", tags="temperature")
    return datastreamDHT2temp



def get_datastreamRH1(feed):
  try:
    datastreamRH1 = feed.datastreams.get("DHT1_RH")
    return datastreamRH1
  except:
    datastreamRH1 = feed.datastreams.create("DHT1_RH", tags="humidity")
    return datastreamRH1

def get_datastreamRH2(feed):
  try:
    datastreamRH2 = feed.datastreams.get("DHT2_RH")
    return datastreamRH2
  except:
    datastreamRH2 = feed.datastreams.create("DHT2_RH", tags="humidity")
    return datastreamRH2


# main program entry point - runs continuously updating our datastream with the
# latest temperature reading
def run():
        iteration = 0
        runflag = True
        runcnt = 0
        l = ser.readline()
        while(runflag):
            while(True):

                try:
                    print "get feed"
                    feed = api.feeds.get(FEED_ID)

                    datastreamTemp1 = get_datastreamTemp(feed)
                    datastreamTemp1.max_value = None
                    datastreamTemp1.min_value = None
                    datastreamTemp2 = get_datastreamTemp2(feed)
                    datastreamTemp2.max_value = None
                    datastreamTemp2.min_value = None


                    datastreamTemp3 = get_datastreamTemp2(feed)
                    datastreamTemp3.max_value = None
                    datastreamTemp3.min_value = None
                    datastreamRH1 = get_datastreamRH1(feed)
                    datastreamRH1.max_value = None
                    datastreamRH1.min_value = None
                    datastreamRH2 = get_datastreamRH2(feed)
                    datastreamRH2.max_value = None
                    datastreamRH2.min_value = None
                    lastTemp = get_datastreamTemp(feed)
                    pass

                except requests.HTTPError as e:
                    print "get datastream error"
                    pass



                l = ser.readline()
                data = l.split(',')

                print l

                humidity1 = data[0]
                temperature1 = data[1]
                humidity2 = data[2]
                temperature2 = data[3]
                humidity3 = data[4]
                temperature3 = data[5]
                print "data"
                print data



                    #print(l)
                    #print  datetime.datetime.utcnow(), bmpPressure, bmpTempF, humidity, bmpTempC

                    # with open(filename, 'w+') as f:
                    #     result = csv.writer(f, delimiter=',', dialect='excel')
                    #     result.writerow(edata)
                    #     myfile.write(edata)
                    # print edata

                    # iteration += 1
                    # if(iteration % 12 == 0):
                    #         print("myfile.flush()")
                    #         myfile.flush()
                    #         os.fsync(myfile.fileno())

                degrees1 = temperature1
                degrees2 = temperature2
                degrees3 = temperature3
                RH1 = humidity1
                RH2 = humidity2
                RH3 = humidity3



                print degrees1, degrees2, degrees3, RH1, RH2, RH3
                datastreamTemp1.current_value = degrees1
                datastreamTemp1.at = datetime.datetime.utcnow()
                datastreamTemp2.current_value = degrees3
                datastreamTemp2.at = datetime.datetime.utcnow()
                datastreamRH1.current_value = RH1
                datastreamRH1.at = datetime.datetime.utcnow()
                datastreamRH2.current_value = RH3
                datastreamRH2.at = datetime.datetime.utcnow()


                try:
                        datastreamTemp1.update()
                        datastreamTemp2.update()
                        datastreamRH1.update()
                        datastreamRH2.update()
                        print "uploading..."
                        #print pressure, RH, degrees
                        readingcnt = 0
			while readingcnt < 150:
				l = ser.readline()
				readingcnt = readingcnt + 1
				print "waiting"
			#time.sleep(300)
                except requests.HTTPError as e:
                        print "HTTPError({0}): {1}".format(e.errno, e.strerror)
                        nowTime = str(datetime.datetime.utcnow())
                        l = ""
                        l+=str(nowTime)
                        l+=", HTTP Error "
                        myfile.write(l)
                        iteration += 1
                        if(iteration % 12 == 0):
                            myfile.flush()
                            os.fsync(myfile.fileno())
                        time.sleep(300)
                        pass
                else:
                    time.sleep(10)
                    break

run()

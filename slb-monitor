#!/usr/bin/python3
#coding=utf-8

from confluent_kafka import Producer
from aliyunsdkcore.client import AcsClient
from aliyunsdkcore.request import CommonRequest
from aliyunsdkcore.auth.credentials import AccessKeyCredential
from aliyunsdkcms.request.v20190101.DescribeMetricListRequest import DescribeMetricListRequest
from aliyunsdkcms.request.v20190101.DescribeMetricMetaListRequest import DescribeMetricMetaListRequest

import json
import datetime

credentials = AccessKeyCredential('XXXXXX', 'XXXXXXX')
client = AcsClient(region_id='cn-beijing', credential=credentials)

p = Producer({'bootstrap.servers':'1.1.1.1:9092,2.2.2.2:9092'})
p.poll(0)

start_time = (datetime.datetime.now()-datetime.timedelta(hours=8)-datetime.timedelta(minutes=3)).strftime("%Y-%m-%dT%H:%M:00Z")
end_time = (datetime.datetime.now()-datetime.timedelta(hours=8)-datetime.timedelta(minutes=2)).strftime("%Y-%m-%dT%H:%M:00Z")
#end_time = (datetime.datetime.now()-datetime.timedelta(hours=9)).strftime("%Y-%m-%dT%H:%M:00Z")

def get_instance_info():
    request = CommonRequest()
    request.set_accept_format('json')
    request.set_domain('slb.aliyuncs.com')
    request.set_method('POST')
    request.set_protocol_type('https')  # https | http
    request.set_version('2014-05-15')
    request.set_action_name('DescribeLoadBalancers')

    response = client.do_action_with_exception(request)
    #response_dic = json.loads(response)  3.6往上
    response_dic = json.loads(str(response, encoding='utf-8'))
    Datapoint = response_dic["LoadBalancers"]["LoadBalancer"]
    return Datapoint

def get_slb_metric():
    request = DescribeMetricMetaListRequest()
    request.set_accept_format('json')
    request.set_Namespace("acs_slb_dashboard")
    request.set_PageSize(100)

    response = client.do_action(request)
    response_dic = json.loads(str(response, encoding='utf-8'))
    Datapoint = response_dic["Resources"]["Resource"]
    return Datapoint
def get_slb_data(metric):
    request = DescribeMetricListRequest()
    request.set_accept_format('json')
    request.set_Namespace("acs_slb_dashboard")
    request.set_MetricName(metric)
    request.set_Period("60")
    request.set_StartTime(start_time)
    request.set_EndTime(end_time)
    request.set_Length("200")
    #request.set_Dimensions({"instanceId" : "xibalaxi"})  #备选

    response = client.do_action(request)
    response_dic = json.loads(str(response, encoding='utf-8'))
    Datapoint = response_dic["Datapoints"]
    return Datapoint


if __name__ == '__main__':

    slb_instance_message= dict()
    slb_instance= get_instance_info()
    for i in range(len(slb_instance)):
        lb_id = slb_instance[i]["LoadBalancerId"]
        lb_name = slb_instance[i]["LoadBalancerName"]
        slb_instance_message[lb_id] = lb_name

    slb_metrics = get_slb_metric()
    for j in range(len(slb_metrics)):
        metric_name = slb_metrics[j]["MetricName"]
        metric_desc = slb_metrics[j]["Description"]

        try:
            slb_data = json.loads(get_slb_data(metric_name))
            for k in range(len(slb_data)):
                instance_vip = slb_data[k]["vip"]
                average_value = slb_data[k]["Average"]
                instance_port = slb_data[k]["port"]
                instance_id = slb_data[k]["instanceId"]
                instance_name = slb_instance_message[instance_id]
                data_analyze = ("kafka {0} SLB {1} {2} {3} {4} {5} {6} {7} lfk").format(end_time, instance_id, instance_name, instance_vip, instance_port, metric_name, metric_desc,average_value)
                p.produce("aliyun-slb-monitor",data_analyze.encode("utf-8"))
                #print(data_analyze)
        except Exception as e:
            pass


# DRF approach:
  
//part/views.py
```python
from rest_framework import viewsets
from rest_framework import filters

from part.models import part_family_table
from part.serializers import PartFamilySerializer
from common_logger import log_error
from common_classes import GenerateResponse
from common_response import NO_CONTENT, NO_CONTENT_CODE, INTERNAL_SERVER_ERROR_CODE, INTERNAL_SERVER_ERROR, CREATED_CODE, RECORD_ADDED

  
class PartFamilyViewSet(viewsets.ModelViewSet):

    """
    This viewset automatically provides `list`, `create`, `retrieve`,
    `update` and `destroy` actions.
    """
    # Initialize and setup the default response_format to the response standard format as defined in the base_format
    def __init__(self, **kwargs):
        self.response_generator = GenerateResponse()
        super(PartFamilyViewSet, self).__init__(**kwargs)

    queryset = part_family_table.objects.all()
    serializer_class = PartFamilySerializer

    # Addendum: Ordering filters along with base filter setting
    filter_backends = [filters.OrderingFilter]
    # Filterable fields
    filterset_fields = ['part_family_name', 'line_id',"shop_id", "plant_id"]
    # Ordering fields
    ordering_fields = ['part_family_name']
    # Default ordered on following fields
    ordering = ['part_family_name']

    def list(self, request, *args, **kwargs):
        try:
            response_from_ListModelMixin = super().list(request, *args, **kwargs)
            data_list = response_from_ListModelMixin.data
            # print(data_list)
            # if count present indicates pagination is applied else bypassed by passing page_size = -1 from query params
            if 'count' in data_list:
                if data_list['count'] == 0:
                    return self.response_generator.generate_response(data=data_list,status=NO_CONTENT_CODE, message=NO_CONTENT)
            return self.response_generator.generate_response(data=data_list)
        except Exception as e:
            log_error(e)
            return self.response_generator.generate_response(error=True, status=INTERNAL_SERVER_ERROR_CODE,message=INTERNAL_SERVER_ERROR, data=None)

    def retrieve(self, request, *args, **kwargs):
        # try:
            response_from_RetrieveModelMixin =super().retrieve(request, *args, **kwargs)
            data_object = response_from_RetrieveModelMixin.data
            return self.response_generator.generate_response(data=data_object)
        # except Exception as e:
        #     log_error(e)
        #     return self.response_generator.generate_response(error=True, status=INTERNAL_SERVER_ERROR_CODE,message=INTERNAL_SERVER_ERROR, data=None)

    def create(self, request, *args, **kwargs):
        # try:
            response_from_CreateModelMixin = super().create(request, *args, **kwargs)
            data_object = response_from_CreateModelMixin.data
            return self.response_generator.generate_response(data=data_object, status=CREATED_CODE, message=RECORD_ADDED)
        # except Exception as e:
        #     log_error(e)
        #     return self.response_generator.generate_response(error=True, status=INTERNAL_SERVER_ERROR_CODE,message=INTERNAL_SERVER_ERROR, data=None)
```

//common_classes.py
```python
from rest_framework.response import Response
from rest_framework.pagination import LimitOffsetPagination

from common_response import DATA_RETRIEVED, OK_CODE

class DefaultCustomPagination(LimitOffsetPagination):
    limit_query_param = 'page_size'
    max_limit = 10000
    
    def paginate_queryset(self, queryset, request, view):
            # Setting page_size query_params to -1 will result in all data and bypass the rest_framework/custom pagination strategy used. Hence, no default wrapper available for response.
            if self.limit_query_param in request.query_params and request.query_params[self.limit_query_param] == "-1":
                return None
            return super().paginate_queryset(queryset, request, view)

class GenerateResponse(object):

    """
    Standard Output Response format.
    """
    def __init__(self, **args):
        self.response_base_format = {
            "status": args.get('status', OK_CODE),
            "error": args.get('error', False),
            "data": args.get('data', []),
            "message": args.get('message', DATA_RETRIEVED)
        }

    def generate_response(self,status=OK_CODE,error=False,data=[],message=DATA_RETRIEVED):
        """
        Generates JSONResponse based on the parameters entered
        """
        self.response_base_format['status'] = status
        self.response_base_format['error'] = error
        self.response_base_format['data'] = data
        self.response_base_format['message'] = message
        return Response(data=self.response_base_format, status=status)
```

//common_functions.py
```python
from django.http import JsonResponse
from rest_framework.views import exception_handler

from common_response import *

def generate_response(status,code=200,message=DATA_RETRIEVED,data=""):

    """
    Generates JSON Response based on the parameters entered
    """

    base_struct = {
        "status":code,
        "result":data,
        "message":message
    }
    
    if not status:
        base_struct["status"]=code
        base_struct["message"]=message
        base_struct["result"]=data
    # Status?
    return JsonResponse(base_struct,status=code)

def validate_object_keys(obj, required_keys):
    return set(required_keys).issubset(obj.keys())

def keep_required_keys(data_dict, required_keys):
    keys_to_delete = [key for key in data_dict.keys() if key not in required_keys]

    for key in keys_to_delete:
        del data_dict[key]
    return data_dict

def custom_exception_handler(exc, context):
    # Call REST framework's default exception handler first,
    # to get the standard error response.
    response = exception_handler(exc, context)
    # Now add the HTTP status code to the response.
    if response is not None:
        response.data['status_code'] = response.status_code
    return response
```


# Non DRF Approach
```python 

#django
from django.urls import reverse
from django.test import RequestFactory
import json
from django.test import RequestFactory
from rest_framework.decorators import api_view
from django.db.models import Q
import arrow
from django.db.models import F
from django.db.models import OuterRef, Subquery

#internal
from common_function import *
from common_response import *
from common_logger import *
from common_variables import *
from trolley.models import *
from part.models import *
from external.models import *
from django.db.models import Q,Subquery, OuterRef

@api_view(['POST'])
def create_part_family(req):
    """
    API Endpoint function to add a part family to part family table
    """
    try:
        req = json.loads(req.body)
        if validate_object_keys(req,PART_FAMILY_ADD_PARAMS):
            req= remove_special_character_spaces(req)
            
            
            #validated if shop_id,line_id, plant_id exist or not 
            plant = plant_table.objects.filter(plant_id__contains=req['plant_id']).exists()
            shop = plant_table.objects.filter(shop_id__contains=[req['shop_id']]).exists()
            line = shop_table.objects.filter(line_id__contains=[req['line_id']]).exists()

            if plant == False:
                return generate_response(False,NOT_FOUND_CODE,PLANT_NOT_FOUND)
            elif shop==False:
                return generate_response(False,NOT_FOUND_CODE,SHOP_NOT_FOUND)
            elif line==False:
                return generate_response(False,NOT_FOUND_CODE,LINE_NOT_FOUND)
            
            unique_check_table = part_family_table.objects.all() 
            filters = {}

            for field in PART_FAMILY_ADD_PARAMS:
                value = req[field]
                if value is not None:
                    filters[field] = value

            if filters:
                q_objects = Q()
                for field, value in filters.items():
                    if value:
                        q_objects &= Q(**{field: value})

                unique_check_table = unique_check_table.filter(q_objects)
            
            unique_check_table = unique_check_table.values()
            # check if the data exist or not
            if unique_check_table:
                # if deleted updates the record with the latest info and changes th deleted status
                if unique_check_table[0]["is_deleted"]:
                    unique_id = unique_check_table[0]["id"]
                    req = keep_required_keys(req,PART_FAMILY_ADD_PARAMS)
                    time_now = get_time_now()
                    part_family_table.objects.filter(id=unique_id).update(**req,is_deleted=False,updated_at=time_now)
                    log_info(RECORD_ADDED)
                    return generate_response(True,message=RECORD_ADDED)
                else:
                    log_warning(RECORD_ALREADY_EXISTS)
                    return generate_response(False,BAD_REQUEST_CODE,RECORD_ALREADY_EXISTS)
                
            #create new data 
            time_now = get_time_now()
            part_family_table.objects.create(**req,updated_at=time_now,created_at=time_now)
            
            
            return generate_response(True,OK_CODE,RECORD_ADDED)
        else:
            log_warning(BAD_REQUEST)
            return generate_response(False,BAD_REQUEST_CODE,BAD_REQUEST)
    except Exception as e:
        log_error(e)
        return generate_response(False,INTERNAL_SERVER_ERROR_CODE,INTERNAL_SERVER_ERROR)
    
@api_view(['POST'])
def view_part_bom(req):
    try:
        req = json.loads(req.body)
        if validate_object_keys(req,BOM_VIEW_REQ_PARAMS):
            req = remove_special_character_spaces(req)
            req = convert_values_to_int(req,["page_size","offset"])
            if not req:
                log_warning(INVALID_NUMBER)
                return generate_response(False,BAD_REQUEST_CODE,INVALID_NUMBER)
            

            bom_queryset = bom_table.objects.all()

            if req["status"] == "complete":
                bom_queryset = bom_queryset.filter(status = True)
            
            if req["status"] == "incomplete":
                bom_queryset = bom_queryset.filter(status = False)

            filter_fields = ['plant_id', 'shop_id', 'line_id']
            filters = {}


            for field in filter_fields:
                value = req[field]
                if value is not None:
                    filters[field] = value

            if filters:
                q_objects = Q()
                for field, value in filters.items():
                    if value:
                        q_objects &= Q(**{field: value})

            bom_queryset = bom_queryset.filter(q_objects)

            if req["sort_by"] != "":
                bom_queryset = bom_queryset.filter(is_deleted = False).order_by(req["sort_by"])
            else:
                bom_queryset = bom_queryset.filter(is_deleted = False).order_by("status")

            # check total count before slicing
            total_count = bom_queryset.count()

            # if no data found for certain filter conditions
            if total_count == 0:
                log_warning(NO_CONTENT)
                return generate_response(False,NO_CONTENT_CODE,NO_CONTENT)
            
            if req["reversed"]:
                bom_queryset = bom_queryset.reverse()

            page_size = int(req["page_size"])
            offset = int(req["offset"])
            if offset != -1 :
                bom_queryset = bom_queryset[offset:offset+page_size]

            bom_queryset = bom_queryset.annotate(
                part_identification = Subquery(part_table.objects.filter(part_no=OuterRef('part_no')).values('part_identification')[:1]),
                part_description = Subquery(part_table.objects.filter(part_no=OuterRef('part_no')).values('part_description')[:1]),
                part_image_link = Subquery(part_table.objects.filter(part_no=OuterRef('part_no')).values('part_image_link')[:1]),
            )
            bom_queryset = list(bom_queryset.values())

            bom_queryset = list(map(unix_to_timestamp_obj,bom_queryset))

            bom_queryset = add_passive_id(bom_queryset)

            data = {
                "data" : bom_queryset,
                "total_count" : total_count
            }
            log_info(DATA_RETRIEVED)
            return generate_response(True,data=data)
        else:
            log_warning(BAD_REQUEST)
            return generate_response(False,BAD_REQUEST_CODE,BAD_REQUEST)
    except Exception as e:
        log_error(e)
        return generate_response(False,INTERNAL_SERVER_ERROR_CODE,INTERNAL_SERVER_ERROR)

@api_view(['POST'])
def create_part_bom(req):
    try:
        req = json.loads(req.body)
        if validate_object_keys(req,PART_BOM_ADD_PARAMS):
            # sanitize the object except the image link URL
            req = remove_special_character_spaces_with_exception(req,["part_image_link"])
            req = convert_values_to_int(req,["quantity_required","part_position"])

            # if invalid integer provided in the req
            if not req:
                log_warning(INVALID_NUMBER)
                return generate_response(False,BAD_REQUEST_CODE,INVALID_NUMBER)

            queryset = line_table.objects.filter(line_id=req["line_id"],is_deleted = False)

            # check if line id is present in Digital Twin plant
            if queryset.count()==0:
                log_warning(LINE_NOT_FOUND)
                return generate_response(False,BAD_REQUEST_CODE,LINE_NOT_FOUND)
            

            if queryset.values()[0]["belongs_to_shop_id"] != req["shop_id"]:
                log_warning(SHOP_NOT_FOUND)
                return generate_response(False,BAD_REQUEST_CODE,SHOP_NOT_FOUND)

            queryset = shop_table.objects.filter(shop_id=req["shop_id"],is_deleted = False)
            
            if queryset.count()==0:
                log_warning(SHOP_NOT_FOUND)
                return generate_response(False,BAD_REQUEST_CODE,LINE_NOT_FOUND)
            
            # check if shop id is present in Digital Twin
            if queryset.values()[0]["belongs_to_plant_id"] != req["plant_id"]:
                log_warning(PLANT_NOT_FOUND)
                return generate_response(False,BAD_REQUEST_CODE,PLANT_NOT_FOUND)


            # check part family details

            queryset = part_family_table.objects.filter(part_family_name = req["part_family_name"],is_deleted = False)

            if queryset.count()==0:
                log_warning(PART_FAMILY_NOT_FOUND)
                return generate_response(False,BAD_REQUEST_CODE,PART_FAMILY_NOT_FOUND)
            

            # check bom for duplicate
            queryset = bom_table.objects.filter(part_no = req["part_no"],vc_id=req["vc_id"],part_family_name=req['part_family_name'])
            if queryset.count()!=0:
                log_warning(DUPLICATE_RECORD_FOUND)
                return generate_response(False,BAD_REQUEST_CODE,DUPLICATE_RECORD_FOUND)

            # check if part no exist in the part db
            queryset = part_table.objects.filter(part_no = req["part_no"])

            # make req for entering data into part table
            req_for_part = keep_required_keys(req,PART_FIELD_PARAMS)

            current_time = get_time_now()
            
            # new part_no found
            if queryset.count()==0:
                log_info(PART_TABLE_ADDED)
                queryset = part_table.objects.create(**req_for_part,created_at=current_time,updated_at=current_time)
            
            #  check if part is deleted .. if deleted then update the part with latest info
            elif queryset.values()[0]["is_deleted"]:
                log_info(PART_TABLE_UPDATED)
                part_table.objects.filter(id=queryset.values()[0]["id"]).update(**req_for_part,updated_at=current_time,is_deleted = False)
                
            else :
                log_info(PART_TABLE_UPDATED)
                part_table.objects.filter(id=queryset.values()[0]["id"]).update(**req_for_part,updated_at=current_time)
            
            
            bom_req = keep_required_keys(req,BOM_CREATE_ENTRY_PARAMS)

            timenow = get_time_now()
            bom_table.objects.create(**bom_req,part_family_name = req["part_family_name"],updated_at=timenow,status=True,created_at=timenow,updated_by=req["user_name"])
            log_info(BOM_TABLE_ADDED)
            return generate_response(True)
        else:
            log_warning(BAD_REQUEST)
            return generate_response(False,BAD_REQUEST_CODE,BAD_REQUEST)
    except Exception as e:
        log_error(e)
        return generate_response(False,INTERNAL_SERVER_ERROR_CODE,INTERNAL_SERVER_ERROR)

@api_view(['POST'])
def delete_part_bom(req):
    try:
        req = json.loads(req.body)
        if validate_object_keys(req,["id"]):
            unique_id = req["id"]
            time_now = get_time_now()
            queryset = bom_table.objects.filter(id=unique_id)
            if queryset.count()!=0:
                queryset.update(is_deleted=True,updated_at=time_now)
            else:
                log_warning(BOM_ENTRY_NOT_FOUND)
                return generate_response(False,NOT_FOUND_CODE,BOM_ENTRY_NOT_FOUND)
            log_info(BOM_TABLE_DELETED)
            return generate_response(True,message=BOM_TABLE_DELETED)
        else:
            log_warning(BAD_REQUEST)
            return generate_response(False,BAD_REQUEST_CODE,BAD_REQUEST)
    except Exception as e:
        log_error(e)
        return generate_response(False,INTERNAL_SERVER_ERROR_CODE,INTERNAL_SERVER_ERROR)

@api_view(['POST'])
def update_part_bom(req):
    try:
        req = json.loads(req.body)
        if validate_object_keys(req,PART_BOM_UPDATE_PARAMS):
            # sanitize the object except the image link URL
            req = remove_special_character_spaces_with_exception(req,["part_image_link"])
            req = convert_values_to_int(req,["quantity_required","part_position","id"])
            
            if not req:
                log_warning(INVALID_NUMBER)
                return generate_response(False,BAD_REQUEST_CODE,INVALID_NUMBER)

            queryset = line_table.objects.filter(line_id=req["line_id"],is_deleted = False)
            # check if line id is present in Digital Twin plant
            if queryset.count()==0:
                log_warning(LINE_NOT_FOUND)
                return generate_response(False,BAD_REQUEST_CODE,LINE_NOT_FOUND)
            

            if queryset.values()[0]["belongs_to_shop_id"] != req["shop_id"]:
                log_warning(SHOP_NOT_FOUND)
                return generate_response(False,BAD_REQUEST_CODE,SHOP_NOT_FOUND)

            queryset = shop_table.objects.filter(shop_id=req["shop_id"],is_deleted = False)
            # check if shop id is present in Digital Twin
            if queryset.values()[0]["belongs_to_plant_id"] != req["plant_id"]:
                log_warning(PLANT_NOT_FOUND)
                return generate_response(False,BAD_REQUEST_CODE,PLANT_NOT_FOUND)


            # check part family details

            queryset = part_family_table.objects.filter(part_family_name = req["part_family_name"],is_deleted = False)

            if queryset.count()==0:
                log_warning(PART_FAMILY_NOT_FOUND)
                return generate_response(False,BAD_REQUEST_CODE,PART_FAMILY_NOT_FOUND)


            # check if part exists
            part_req = keep_required_keys(req,PART_FIELD_PARAMS)
            part_queryset = part_table.objects.filter(part_no = req["part_no"])
            
            current_time = get_time_now()
            
            # new part_no found
            if part_queryset.count()==0:
                log_info(PART_TABLE_ADDED)
                part_queryset = part_table.objects.create(**part_req,created_at=current_time,updated_at=current_time)
            
            #  check if part is deleted .. if deleted then update the part with latest info
            elif part_queryset.values()[0]["is_deleted"]:
                log_info(PART_TABLE_UPDATED)
                part_table.objects.filter(id=part_queryset.values()[0]["id"]).update(**part_req,updated_at=current_time,is_deleted = False)

            else :
                log_info(PART_TABLE_UPDATED)
                part_table.objects.filter(id=part_queryset.values()[0]["id"]).update(**part_req,updated_at=current_time)


            # check if duplicate exist in bom table
            queryset = bom_table.objects.filter(
                part_no=req["part_no"],
                vc_id=req["vc_id"],
                part_family_name=req['part_family_name'],
                is_deleted=False
            )
            if queryset.count()!=0:
                if req["id"] != queryset.values()[0]["id"]:
                    log_warning(RECORD_ALREADY_EXISTS)
                    return generate_response(False,BAD_REQUEST_CODE,DUPLICATE_RECORD_FOUND)
                else :
                    req_for_bom = keep_required_keys(req,BOM_CREATE_ENTRY_PARAMS)
                    bom_table.objects.filter(id=req["id"]).update(**req_for_bom,updated_at=get_time_now(),updated_by=req["user_name"])
                    log_info(BOM_TABLE_UPDATED)
                    return generate_response(True,message=BOM_TABLE_UPDATED)

            req_for_bom = keep_required_keys(req,BOM_CREATE_ENTRY_PARAMS)
            bom_table.objects.filter(id=req["id"]).update(**req_for_bom,updated_at=get_time_now(),status=True,updated_by=req["user_name"])
            log_info(BOM_TABLE_UPDATED)
            return generate_response(True,message=BOM_TABLE_UPDATED)
        else:
            log_warning(BAD_REQUEST)
            return generate_response(False,BAD_REQUEST_CODE,BAD_REQUEST)
    except Exception as e:
        log_error(e)
        return generate_response(False,INTERNAL_SERVER_ERROR_CODE,INTERNAL_SERVER_ERROR)

@api_view(['POST'])
def get_part_family(req):
    """
      Api endpoint function to list Part family
    """
    try:
        req = json.loads(req.body)
        if validate_object_keys(req,PART_FAMILY_VIEW_PARAMS):
            
            # check if field exist in model for sorting
            if req["sort_by"] != "" and req["sort_by"] not in [x for x in PART_FAMILY_RESPONSE_PARAMS]:
                log_warning(BAD_REQUEST)
                return generate_response(False,BAD_REQUEST_CODE,BAD_REQUEST)
            
            # check if all page size and offset values are proper
            if int(req["page_size"]) <= 0 or int(req["offset"]) < -1:
                log_warning(BAD_REQUEST)
                return generate_response(False,BAD_REQUEST_CODE,BAD_REQUEST)
            else:
                page_size = int(req["page_size"])
                offset = int(req["offset"])
            

            part_list = part_family_table.objects.annotate(
                        shop_name=Subquery(
                            shop_table.objects.filter(shop_id=OuterRef('shop_id')).values('shop_name')[:1]
                    ),
                        plant_name=Subquery(
                            plant_table.objects.filter(plant_id=OuterRef('plant_id')).values('plant_name')[:1]
                    ),
                        line_name=Subquery(
                            line_table.objects.filter(line_id=OuterRef('line_id')).values('line_name')[:1]
                    )
                    )
            
            
            #check for the filter in by shop id and line id
            if req['shop_id']!='' and req['line_id']!='':
                part_list = part_list.filter(shop_id=req['shop_id'],line_id=req['line_id'],is_deleted=False).values(*PART_FAMILY_FILTER_TABLE)
            else:
                part_list = part_list.filter(is_deleted=False).values(*PART_FAMILY_FILTER_TABLE)
            

            part_count = part_list.count()

            if req["sort_by"] != "":
                part_list = part_list.order_by(req['sort_by'])
            else:
                part_list = part_list.order_by("part_family_name")

                # part_list = sorted(part_list,key=lambda x: x['part_family_name'])

            part_list=list(part_list)

            if part_count == 0:
                log_warning(NO_CONTENT)
                return generate_response(False,NO_CONTENT_CODE,NO_CONTENT)
            if req['reversed']:
                part_list.reverse()
            
            
            #checking the offset
            if offset != -1 :
                part_list = part_list[offset:offset+page_size]

            part_list= add_passive_id(part_list) #added passive id to the data 

            for parts in part_list:
                for flag in PART_FAMILY_FLAG:
                    if parts[flag]==True:
                        parts[flag]= 'Enable'
                    else:
                        parts[flag]= 'Disable'
                parts['updated_at']=unix_to_datetime(float(parts['updated_at'])) #covert unix time format to Format the datetime with AM/PM representation

            #formatting the data
            data = {
                'data':part_list,
                'total_count':part_count
            }
            
            log_info(DATA_RETRIEVED)
            return generate_response(True,message=DATA_RETRIEVED,data=data)
        else:
            log_warning(BAD_REQUEST)
            return generate_response(False,BAD_REQUEST_CODE,BAD_REQUEST)
    except Exception as e:
    
        log_error(e)
        return generate_response(False,INTERNAL_SERVER_ERROR_CODE,INTERNAL_SERVER_ERROR)

@api_view(['POST'])
def update_part_family(req):
    """
    Api endpoint to update the part family
    """
    try:
        req = json.loads(req.body)
        if validate_object_keys(req,PART_FAMILY_UPDATE_PARAMS):

            req= remove_special_character_spaces(req)

            plant = plant_table.objects.filter(plant_id__contains=req['plant_id']).exists()
            shop = plant_table.objects.filter(shop_id__contains=[req['shop_id']]).exists()
            line = shop_table.objects.filter(line_id__contains=[req['line_id']]).exists()
            if plant & shop & line ==False:
                return generate_response(False,NOT_FOUND_CODE,RECORD_NOT_FOUND)
            
            # Filter process to check weather same record is already present in db
            record_to_update = part_family_table.objects.all()
            filter_fields = PART_FAMILY_ADD_PARAMS
              
            filters = {}

            for field in filter_fields:
                value = req[field]
                if value is not None:
                    filters[field] = value

            if filters:
                q_objects = Q()
                for field, value in filters.items():
                    if value:
                        q_objects &= Q(**{field: value})
                
                record_to_update = record_to_update.filter(q_objects)

            record_to_update = record_to_update.values()
            

            # if record already present checks weather its deleted or not
            if record_to_update:
                # if deleted updates the record with the latest info and changes th deleted status
                if record_to_update[0]["is_deleted"]:
                    log_warning(RECORD_ALREADY_DELETED)
                    return generate_response(False,BAD_REQUEST_CODE,RECORD_ALREADY_DELETED)
                else:
                    # check if the change in details is similar to some other part
                    if str(record_to_update[0]["id"]) != str(req["id"]):
                        log_warning(DUPLICATE_RECORD_FOUND)
                        return generate_response(False,BAD_REQUEST_CODE,DUPLICATE_RECORD_FOUND)
                    else:
                        # updates info with changes other than plant , line , shop and part family  name
                        unique_id = record_to_update[0]["id"]
                        req = keep_required_keys(req,PART_FAMILY_ADD_PARAMS)
                        time_now = get_time_now()
                        part_family_table.objects.filter(id=unique_id).update(**req,is_deleted=False,updated_at=time_now)
                        log_info(RECORD_UPDATE_DONE)
                        return generate_response(True,message=RECORD_UPDATE_DONE)

            else:
                #update if no similar data found
                unique_id = req["id"]
                req = keep_required_keys(req,PART_FAMILY_ADD_PARAMS)
                time_now = get_time_now()
                part_family = part_family_table.objects.exclude(id=unique_id)
                is_exist = part_family.filter(part_family_name=req['part_family_name']).exists()
                if is_exist:
                    return generate_response(False,BAD_REQUEST_CODE,PART_FAMILY_DETAILS_ALREADY_EXIST)
                else:
                    part_family_table.objects.filter(id=unique_id).update(**req,is_deleted=False,updated_at=time_now)
                    

                log_info(RECORD_UPDATE_DONE)
                return generate_response(True,message=RECORD_UPDATE_DONE)
        else:
            log_warning(BAD_REQUEST)
            return generate_response(False,message=BAD_REQUEST,code=BAD_REQUEST_CODE)
                 
    except Exception as e:
        log_error(e)
        return generate_response(False,INTERNAL_SERVER_ERROR_CODE,INTERNAL_SERVER_ERROR)
    

@api_view(['POST'])
def delete_part_family(req):
    """
    Api view function to delete part family from part family table
    """
    try:
        req = json.loads(req.body)
        if validate_object_keys(req,["id"]):
            for id in req['id']:
                part = part_family_table.objects.filter(id=id).values()
                if part.count() ==0:
                    return generate_response(False,message=PART_FAMILY_NOT_FOUND,code=BAD_REQUEST_CODE)
                if part[0]['is_deleted']==True:
                    return generate_response(False,message=BAD_REQUEST,code=BAD_REQUEST_CODE)
                else:
                    time_now = get_time_now()
                    part.update(is_deleted=True,updated_at = time_now)

            return generate_response(True,message=PART_DELETED_SUCCESS)
        else:
            log_warning(BAD_REQUEST)
            return generate_response(False,message=BAD_REQUEST,code=BAD_REQUEST_CODE)
    except Exception as e:
        log_error(e)
        return generate_response(False,INTERNAL_SERVER_ERROR_CODE,INTERNAL_SERVER_ERROR)

@api_view(['POST'])
def bulk_create_part_bom(req):
    '''
    Api view function to add bulk data in BOM
    '''
    try:
        req = json.loads(req.body)
        factory = RequestFactory()

        bulk_data_list = req['data']
        
        if len(bulk_data_list) !=0:
            data_success_count = 0
            data_failed_count = 0
            data_failed = []
            if not validate_object_keys(req,["plant_id","data","user_name"]):
                log_warning(PLANT_NOT_FOUND)
                return generate_response(False,BAD_REQUEST_CODE,PLANT_NOT_FOUND)
            else:
                plant = req["plant_id"]
                user_name = req["user_name"]
            for data in bulk_data_list:
                data["plant_id"]=plant
                data["user_name"]=user_name
                mock_request = factory.post('create-part-bom', data=data, content_type='application/json')
                response = create_part_bom(mock_request)
                response_data = json.loads(response.getvalue().decode())

                if response_data['status']==200:
                    data_success_count +=1
                else:
                    data_failed_count +=1
                    data['failed_message']= response_data['message']
                    data_failed.append(data)
            
            result = {
                "success_count": data_success_count,
                "failed_count": data_failed_count,
                "failed_data": data_failed
            }
            return generate_response(True,OK_CODE,DATA_RETRIEVED,data=result)
        else:
                return generate_response(False,BAD_REQUEST_CODE,BAD_REQUEST)
    
    except Exception as e:
        log_error(e)
        return generate_response(False,INTERNAL_SERVER_ERROR_CODE,INTERNAL_SERVER_ERROR)

@api_view(['POST'])
def bulk_delete_bom(req):
    '''
    Api view function to add bulk data in BOM
    '''
    try:

        req = json.loads(req.body)
        factory = RequestFactory()

        if "id" not in req and type(req["id"]) != "list":
            log_warning(BAD_REQUEST)
            return generate_response(False,BAD_REQUEST_CODE,BAD_REQUEST)
        
        bulk_data_list = req["id"]

        if len(bulk_data_list) !=0:
            data_success_count = 0
            data_failed_count = 0
            data_failed = []
            for data in bulk_data_list:
                mod_data = {"id":data}
                mock_request = factory.post('create-part-bom', data=mod_data, content_type='application/json')
                response = delete_part_bom(mock_request)
                response_data = json.loads(response.getvalue().decode())
                if response_data['status']==200:
                    data_success_count +=1
                else:
                    data_failed_count +=1
                    mod_data['failed_message'] = response_data['message']
                    data_failed.append(mod_data)
            
            result = {
                "success_count": data_success_count,
                "failed_count": data_failed_count,
                "failed_data": data_failed
            }
            return generate_response(True,OK_CODE,DATA_RETRIEVED,data=result)
        else:
                return generate_response(False,BAD_REQUEST_CODE,BAD_REQUEST)
    
    except Exception as e:
        log_error(e)
        return generate_response(False,INTERNAL_SERVER_ERROR_CODE,INTERNAL_SERVER_ERROR)

@api_view(['POST'])
def create_bulk_part_family(req):
    """
    Api view function to add bulk data in part family data
    """
    try:
        req = json.loads(req.body)

        factory = RequestFactory()

        bulk_data_list = req['data']
        if len(bulk_data_list) !=0:
            data_success_count = 0
            data_failed_count = 0
            data_failed = []
            for data in bulk_data_list:

                #create request for add data 
                data['plant_id']=req['plant_id']
                mock_request = factory.post('create-part-family', data=data, content_type='application/json')

                response = create_part_family(mock_request) #call the create function 
                response_data = json.loads(response.getvalue().decode())
                if response_data['status']==200:
                    data_success_count +=1
                else:
                    data_failed_count +=1
                    data['failed_message']= response_data['message']
                    data_failed.append(data)
            
            result = {
                "success_count": data_success_count,
                "failed_count": data_failed_count,
                "failed_data": data_failed
            }
            return generate_response(True,OK_CODE,SUCCESS,data=result)
        else:
            return generate_response(False,message="Invalid Data")

    except Exception as e:
        log_error(e)
        return generate_response(False,INTERNAL_SERVER_ERROR_CODE,INTERNAL_SERVER_ERROR)

@api_view(['POST'])
def add_skip_configuration(req):
    try:
        req = json.loads(req.body)
        if validate_object_keys(req,SKIP_CONFIGURATION_ADD_PARAMS):
            message = req['message']
            plant_exist = plant_table.objects.filter(plant_id=req['plant_id']).exists()
            if plant_exist==False:
                return generate_response(False,NOT_FOUND_CODE,PLANT_NOT_FOUND)
            for data in message:
                message = skip_configuration_table.objects.filter(skip_message=data).values()
                if message.exists():
                    if message[0]['is_deleted']:
                        skip_configuration_table.objects.filter(skip_message=data).update(is_deleted=False,updated_by=req['created_by'],updated_at=get_time_now(),plant_id=req['plant_id'])
                else:
                    skip_configuration_table.objects.create(skip_message=data,created_by=req['created_by'],updated_by=req['created_by'],created_at=get_time_now(),updated_at=get_time_now(),plant_id=req['plant_id'])
                
            return generate_response(True,message=RECORD_ADDED)
        else:
            return generate_response(False,message=INVALID_DATA,code=400)
    except Exception as e:
        log_error(e)
        return generate_response(False,INTERNAL_SERVER_ERROR_CODE,INTERNAL_SERVER_ERROR)
    
@api_view(['POST'])
def get_skip_configuration(req):
    """
    Api view function to list skip configuration messages
    """
    try:
        req=json.loads(req.body)
        if validate_object_keys(req,SKIP_MESSAGE_VIEW_PARAMS):
            message = skip_configuration_table.objects.filter(plant_id=req['plant_id']).values('skip_message','skip_config_id')
            if message.count()==0:
                return generate_response(False,NOT_FOUND_CODE,RECORD_NOT_FOUND)
            else:
                message_count = message.count()
                message_list = [{'id':data['skip_config_id'],'value':data['skip_message']} for data in message]
                data = {
                    'data':message_list,
                    'count':message_count
                }
                return generate_response(True,OK_CODE,data=data)
        else:
            return generate_response(False,BAD_REQUEST_CODE,BAD_REQUEST)
    except Exception as e:
        log_error(e)
        return generate_response(False,INTERNAL_SERVER_ERROR_CODE,INTERNAL_SERVER_ERROR)
    
@api_view(['POST'])
def view_sequence_report(req):
    """
    API Endpoint function to view VIN sequencing report against plant, shops and lines
    """
    try:
        
        req = json.loads(req.body)
        if validate_object_keys(req, SEQUENCE_REPORT_PARAMS):
            req=remove_special_character_spaces(req)
            
            if req["sort_by"] != "" and req["sort_by"] not in [x.name for x in vehicle_part_completion_tracker_table._meta.get_fields()]:
                log_warning(BAD_REQUEST)
                return generate_response(False,BAD_REQUEST_CODE,BAD_REQUEST)
            
            if is_valid_datetime(req['start_date']) and is_valid_datetime(req['end_date']) and arrow.get(req['start_date'])>arrow.get(req['end_date']):
                log_warning(BAD_REQUEST)
                return generate_response(False,BAD_REQUEST_CODE,BAD_REQUEST)

            # check if all page size and offset values are proper
            if int(req["page_size"]) <= 0 or int(req["offset"]) < -1:
                log_warning(BAD_REQUEST)
                return generate_response(False,BAD_REQUEST_CODE,BAD_REQUEST)
            else:
                page_size = int(req["page_size"])
                offset = int(req["offset"])



            queryset = vehicle_part_completion_tracker_table.objects.all()

            filter_fields = ['plant_id', 'shop_id', 'line_id']  
            filters = {}

            for field in filter_fields:
                value = req[field]
                if value is not None:
                    filters[field] = value

            if filters:
                q_objects = Q()
                for field, value in filters.items():
                    if value:
                        q_objects &= Q(**{field: value})

                queryset = queryset.filter(q_objects)

            
            if req['start_date']!='' and req['end_date']!='':
                start_date = datetime_to_unix(req['start_date'])
                end_date = datetime_to_unix(req['end_date'])

                queryset = queryset.filter(vin_generation_time__range=(start_date,end_date))

            # checks total count before any slicing
            total_count = queryset.count()

            # if no data found for certain filter conditions
            if total_count == 0:
                log_warning(NO_CONTENT)
                return generate_response(False,NO_CONTENT_CODE,NO_CONTENT)
            if req["sort_by"] != "":
            # remove all is deleted records
                queryset = queryset.order_by(req["sort_by"])
            else:
                queryset = queryset.order_by("drop_sequence")

            if req["reversed"]:
                queryset = queryset.reverse()

            # check if offset is for download or viewing
            if offset != -1 :
                queryset = queryset[offset:offset+page_size]



            queryset = list(queryset.values())
            queryset = add_passive_id(queryset)

            for vehicle in queryset:
                vehicle['vin_generation_time'] = unix_to_datetime(int(vehicle['vin_generation_time']))
                vehicle['vin_completion_time'] = unix_to_datetime(int(vehicle['vin_completion_time']))
            
            queryset = add_passive_id(queryset)
            data = {
                "data":queryset,
                "total_count":total_count
            }
            return generate_response(True,data=data)
        else:
            log_warning(BAD_REQUEST)
            return generate_response(False, BAD_REQUEST_CODE, BAD_REQUEST)
    except Exception as e:
        log_error(e)
        return generate_response(False, INTERNAL_SERVER_ERROR_CODE, INTERNAL_SERVER_ERROR)
    
@api_view(['POST'])
def view_familywise_coverage(req):
    """
    API Endpoint function to view VIN sequencing report against plant, shops and lines
    """
    try:
        
        req = json.loads(req.body)
        if validate_object_keys(req, FAMILY_WISE_COVERAGE_PARAMS):
            req=remove_special_character_spaces(req)
            
            if req['vin'] == "":
                log_warning(BAD_REQUEST)
                return generate_response(False,BAD_REQUEST_CODE,BAD_REQUEST)
            if req["sort_by"] != "" and req["sort_by"] not in [x.name for x in vehicle_part_completion_tracker_table._meta.get_fields()]:
                log_warning(BAD_REQUEST)
                return generate_response(False,BAD_REQUEST_CODE,BAD_REQUEST)
            
            # check if all page size and offset values are proper
            if int(req["page_size"]) <= 0 or int(req["offset"]) < -1:
                log_warning(BAD_REQUEST)
                return generate_response(False,BAD_REQUEST_CODE,BAD_REQUEST)
            else:
                page_size = int(req["page_size"])
                offset = int(req["offset"])

            # create list of fields that would be required

            queryset = vehicle_part_completion_tracker_table.objects.all()
            filter_fields = ['plant_id', 'shop_id', 'line_id','vin']  
            filters = {}
 
            for field in filter_fields:
                value = req[field]
                if value is not None:
                    filters[field] = value

            if filters:
                q_objects = Q()
                for field, value in filters.items():
                    if value:
                        q_objects &= Q(**{field: value})

                queryset = queryset.filter(q_objects)

            # checks total count before any slicing
            total_count = queryset.count()

            # if no data found for certain filter conditions
            if total_count == 0:
                log_warning(NO_CONTENT)
                return generate_response(False,NO_CONTENT_CODE,NO_CONTENT)

            if req["sort_by"] != "":
            # remove all is deleted records
                queryset = queryset.order_by(req["sort_by"])
            else:
                queryset = queryset.order_by("drop_sequence")

            # if reversed is true
            if req["reversed"]:
                queryset.reverse()

            # check if offset is for download or viewing
            if offset != -1 :
                queryset = queryset[offset*page_size:offset+page_size]

            fields = [f.name for f in vehicle_part_completion_tracker_table._meta.get_fields()]
            fields.remove("is_deleted")

            queryset = queryset.values(*fields)

            queryset = list(queryset)

            for vehicle in queryset:
                vehicle['vin_generation_time'] = unix_to_datetime(vehicle['vin_generation_time'])
                vehicle['vin_completion_time'] = unix_to_datetime(vehicle['vin_completion_time'])
                vehicle['family_completion_time'] = unix_to_datetime(vehicle['family_completion_time'])

            queryset = add_passive_id(queryset)

            data = {
                "data":queryset,
                "total_count":total_count
            }
            return generate_response(True,data=data)
        else:
            log_warning(BAD_REQUEST)
            return generate_response(False, BAD_REQUEST_CODE, BAD_REQUEST)
    except Exception as e:
        log_error(e)
        return generate_response(False, INTERNAL_SERVER_ERROR_CODE, INTERNAL_SERVER_ERROR)

```
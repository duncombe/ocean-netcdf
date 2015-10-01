# -*- coding: utf-8 -*-
"""
Test script
"""
from seabird.cnv import fCNV
from sys import argv
from Scientific.IO.NetCDF import NetCDFFile as Dataset
import time
from numpy import dtype
from datetime import datetime, timedelta
import csv
import yaml


script, filename = argv

prof = fCNV(filename)

# lets create cf compliant netcdf file

# open a new netCDF file for writing.
# new filename with *.nc
f_name = filename.split('.') 
ncfile = Dataset(f_name[0]+'.nc','w')

#################### Global attributes
#Discovery and identification
 
# create the z/profile dimensions.
ncfile.createDimension('profile',1)
ncfile.createDimension('z',prof['PRES'].data.size)

######################### VARIABLES
#profile
profiles = ncfile.createVariable('profile',dtype('int32').char,('profile',))
profiles.standard_name = 'profile'
profiles.long_name = 'Unique identifier for each instance'
profiles.cf_role = 'profile_id'
profiles[:] = 1

# time
times = ncfile.createVariable('time',dtype('float64').char,('profile',))
times.standard_name = 'time'
times.calendar = 'julian'
times.units = 'seconds since 2000-01-01T00:00:00Z'
times._FillValue = -9.99e-29
times.ancillary_variables = ''
times.axis = 'T'

def datetime2matlabdn(dt):
    mdn = dt + timedelta(days = 366)
    frac = (dt-datetime(dt.year,dt.month,dt.day,dt.hour,dt.minute,dt.second)).seconds / (24.0 * 60.0 * 60.0)
    return mdn.toordinal() + frac

t = datetime2matlabdn(prof.attributes['datetime'])
print t
times[:] = t

# lat
lat = ncfile.createVariable('lat',dtype('float64').char,('profile',))
lat.units = 'degrees_north'
lat.standard_name = 'latitude'
lat.valid_min = -90.0
lat.valid_max = 90.0
lat.axis = 'Y'
lat._Fillvalue = -9.99e-29
lat.ancillary_variables = ''
lat[:] = prof.attributes['LATITUDE']

# lon
lon = ncfile.createVariable('lon',dtype('float64').char,('profile',))
lon.units = 'degrees_east'
lon.standard_name = 'longitude'
lon.valid_min = -180.0
lon.valid_max = 180.0
lon.axis = 'X'
lon._Fillvalue = -9.99e-29
lon.ancillary_variables = ''
lon[:] = prof.attributes['LONGITUDE']

# z
z = ncfile.createVariable('z',dtype('float64').char,('z',))
z.units = 'meter'
z.standard_name = 'altitude'
z.axis = 'Z'
z.positive = "down"
z.valid_min = 0
z.valid_max = 12000.0
z._Fillvalue = -9.99e-29
z.ancillary_variables = ''
z[:] = prof['PRES'].data


variables = prof.keys()

fields = ['parameter name', 'is cf parameter', 'standard/long name', 'units of measurement', 'data code', 'fillValue', 'validMin', 'validMax', 'data_type']

with open('/home/rroman/Desktop/python/imosParameters.txt','rb') as f:
	for i in range(1,15):
		f.next()

	reader = csv.DictReader(f, delimiter=',', fieldnames=fields)

	result ={}
	for row in reader:
	    key = row.pop('parameter name')
	    if key in result:
	        # implement your duplicate row handling here
	        pass
	    result[key] = row

#print float(result['TEMP']['validMin'])
#remove duplicate variable names
variables = list(set(variables))

for item in variables:
	if item in result:
		var = ncfile.createVariable(item,dtype('float64').char,('profile','z',))
		var.longname = result[item]['standard/long name'].strip()
		var.standard_name = result[item]['standard/long name'].strip()
		var.units = result[item]['units of measurement'].strip()
		var._Fillvalue = float(result[item]['fillValue'].strip())
		v = result[item]['validMin'].strip()
		if v:
			var.valid_min = float(v)
			var.valid_max = float(result[item]['validMax'])
		var.platform = 'Seabird911 CTD'
		var.source = 'Sea-Bird SBE 9 Data File'
		var.coordinates ='time lat lon z'
		var[:]=prof[item].data
		C = result[item]['is cf parameter']
		if result[item]['is cf parameter'].strip() == '1':
			var.ancillary_variables = item+'_QC'
			var_q = ncfile.createVariable(item+'_QC',dtype('byte').char,('profile','z',))
			var_q.longname = 'quality_flag of '+item
			var_q.flag_values = 0,1,2,3,4,5,6,7,8,9
			var_q.flag_meanings = 'unknown good_data probably_good_data potentially_correctable bad_data \
	    		bad_data nominal_value interpolated_value missing_value'
			var_q.references = 'OceanSITES QC Flags'		
			var_q[:]= 0

    
# global attributes
fields = ['data_type', 'parameter']

with open('/home/rroman/Desktop/python/global_attributes_profile.txt','rb') as f:

	reader = csv.DictReader(f, delimiter=',',fieldnames=fields)
	global_v = {}
	for row in reader:
	    D = row.pop('data_type')
	    P = row.pop('parameter')
	    K = P.strip().split(':')
	    #key = row.pop('parameter')
	    if K[0] in result:
	        # implement your duplicate row handling here
	        pass
	    global_v[K[0].strip()] = K[1]

ncfile.Conventions = global_v['Conventions']
ncfile.Metadata_Conventions = 'Unidata Dataset Discovery v1.0'
ncfile.featureType = global_v['featureType']
ncfile.cdm_data_type = 'Profile'
ncfile.nodc_template_version = 'NODC_NetCDF_Profile_Template_v1.1'
ncfile.standard_name_vocabulary = global_v['standard_name_vocabulary']


setattr(ncfile, 'site_code', global_v['site_code']) 
setattr(ncfile, 'platform_code', global_v['platform_code'])
setattr(ncfile, 'platform_variable', global_v['platform_variable'])
#setattr(ncfile, 'wmo_platform_code', 'University of Cape Town')
setattr(ncfile, 'title', global_v['title'])
#setattr(ncfile, 'summary', 'University of Cape Town CTD data')
setattr(ncfile, 'naming_authority', global_v['naming_authority'])
#setattr(ncfile, 'id', 'University of Cape Town')

setattr(ncfile, 'source', prof.attributes['filename']) 
setattr(ncfile, 'institution', global_v['institution'])
setattr(ncfile, 'project', global_v['project'])
setattr(ncfile, 'instrument', global_v['instrument'])
setattr(ncfile, 'principle_investigator', global_v['principal_investigator'])

setattr(ncfile, 'creator_name', global_v['creator_name'])
setattr(ncfile, 'creator_email', global_v['creator_email'])

setattr(ncfile, 'cruise', global_v['cruise'])
setattr(ncfile, 'station', global_v['station'])
setattr(ncfile, 'data_restriction', 'none')
setattr(ncfile, 'data_centre', global_v['data_centre'])
setattr(ncfile, 'data_centre_email', global_v['data_centre_email'])
setattr(ncfile, 'comment', global_v['comment'])
setattr(ncfile, 'history', global_v['history'])
setattr(ncfile, 'geospatial_lat_min', prof.attributes['LATITUDE'])
setattr(ncfile, 'geospatial_lat_max', prof.attributes['LATITUDE'])
setattr(ncfile, 'geospatial_lon_min', prof.attributes['LONGITUDE'])
setattr(ncfile, 'geospatial_lon_max', prof.attributes['LONGITUDE'])
setattr(ncfile, 'geospatial_vertical_min', min(prof['DEPTH'].data))
setattr(ncfile, 'geospatial_vertical_max', max(prof['DEPTH'].data))
setattr(ncfile, 'time_coverage_start', prof.attributes['start_time'])
setattr(ncfile, 'time_coverage_end', 'University of Cape Town test program')
setattr(ncfile, 'sea_name', global_v['sea_name'])
setattr(ncfile, 'processing_level', global_v['quality_control_log'])
setattr(ncfile, 'keywords', prof.keys())
setattr(ncfile, 'lineage', global_v['lineage'])
setattr(ncfile, 'uuid', prof.attributes['md5'])
setattr(ncfile, 'license', global_v['license'])
setattr(ncfile, 'date_created', time.strftime("%c"))
setattr(ncfile, 'date_modified', time.strftime("%c"))




# close the file and end the program.
ncfile.close()

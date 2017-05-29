<code>
index=* sourcetype=clamav | rex "(?m-s)^((?<file_path>.+(\\\|\/)))?(?<file_name>[^,\r\n]+): ((?<category>\S+)\s?)?(?<vendor_action>FOUND|OK|moved to|Removed|copied to)\.?$" 
| eventstats values(vendor_action) AS vendor_action_actual values(category) AS category_actual BY file_path file_name 
| stats first(source) AS orig_source values(vendor_action_actual) AS vendor_action BY category file_path file_name host _time 
| lookup clamav_actions vendor_action OUTPUT action 
| eval action=mvjoin(action,","), vendor_action=mvjoin(vendor_action, ",") 
| rename category_actual AS signature
| collect sourcetype=clamav:summary:malware
</code>

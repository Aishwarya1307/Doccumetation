# Raw Scan

**Step 1** 

   
-   When box insert into the Tunnel then , scannig process starts and scanned tags are added in redis_database.
-   https://github.com/iam-rnd/tagid-iot-gateway/blob/82f5f3b3aa0223397730e94649304a2ee3e2062a/src/routes/SocketManager/CrimsouneClub/socket_manager.py#L72
-   
   ![Screenshot from 2023-11-02 17-16-24](https://github.com/Aishwarya1307/Doccumetation/assets/125255809/686f2714-fb57-4135-9274-69d28667586e)


-   When box insert into the Tunnel then send json to UI `{
                    "status": "BOX_INSIDE_TUNNEL",
                    "data": []
                }` 

-   Getting scanned tags data from redis as well as maintaining unique epc's.
-   The `sgtin96_decode` function,decodes the scanned epc's.
-   In `barcode_data` variable, stores a json '{"barcode1":[epc1,epc2,epc3] ,"barcode2":[epc4,epc5,epc6]}' with unique epc's and barcodes.
-   https://github.com/iam-rnd/tagid-iot-gateway/blob/82f5f3b3aa0223397730e94649304a2ee3e2062a/src/routes/SocketManager/CrimsouneClub/socket_manager.py#L103
   
      ![Screenshot from 2023-11-02 17-41-32](https://github.com/Aishwarya1307/Doccumetation/assets/125255809/cc215914-eec5-47df-9fe0-9884c964ffc7)


-   The `compare_epc_logic` function get an entry from postgres database of unique sub_reference_number, then store in `com_met_data` variable

    **comparing epc's**
    
    https://github.com/iam-rnd/tagid-iot-gateway/blob/82f5f3b3aa0223397730e94649304a2ee3e2062a/src/routes/SocketManager/CrimsouneClub/socket_manager_utils.py#L486
    
-   The `process_raw_scan_data` function compare the epc logic and calculate epc count.
-   Parameters :-
  
                  `method` = 'RAW_SCAN'
                  `sub_ref_no` = (unique box number using uuid library)
                  `barcode_data` = { "barcode1":[epc1,epc2,epc3], "barcode2" : [epc4,epc5,epc6] }
    
-
       - `barcodes_` stores unique barcode list
       - `raw_scan_data` stores json, unique barcodes as key and list of epc's as values. Example:- {
                "epc": [epc1,epc2,epc3],
                "count": len(barcode_data[barcode]),
                "product_code": barcode
            }
          - `epc_' list of unique epc's.

 ![Screenshot from 2023-11-02 18-25-32](https://github.com/Aishwarya1307/Doccumetation/assets/125255809/c927ba0f-e539-4b00-8318-cf59ead0d9a3)



-   If `com_met_data`(database entry for current sub_ref_no) is empty then add entry in database in 'raw_scan_metadata' table
-   Else getting data from `com_met_data`

            raw_scan_barcodes_ = com_met_data[0].barcode  ...(list of scanned barcodes)
            raw_scan_epcs_ = com_met_data[0].epc  ...( List of scanned Epcs)
            raw_scan_metadata_ = com_met_data[0].raw_scan_metadata  ... ( json like {"barcode" : {"epc":[],"count":0,"barcode":}})

  - **Compare Data with com_met_data**

    -  Getting an epc from epc_ after that, check epc is present in the `raw_scan_epcs_` .
       -  If it is not present
          -  get 'barcode' from 'barcode_data' ({ "barcode1":[epc1,epc2,epc3], "barcode2" : [epc4,epc5,epc6] })
            -  Check 'epc' is present in the list of epcs of 'barcode',
              - If it is present
                -  append 'epc' in the `raw_scan_epcs_` .
                -  Check barcode from barcode_data is present in 'raw_scann_barcodes_' (stored scanned list barcodes in db)
                   -  barcode is not present
                      -   Add barcode from barcode_data to the 'raw_scan_barcodes_'
                      -   And add json in `raw_scan_metadata_`
                       
                                raw_scan_metadata_[barcode] = {
                                                                  "epc" : [epc],  ...(list of epc's)
                                                                  "count" : 1 ,
                                                                  "product_code" : barcode
                                                                }  
                   -  barcode is present 'raw_scann_barcodes_' 
                      - Add EPC in Barcode Mapping and Increase the Count

                               raw_scan_metadata_[barcode]["epc"].append(ep).....(epc append in list of raw_scan_metadata_[barcode]["epc"])
                               raw_scan_metadata_[barcode]["count"] = raw_scan_metadata_[barcode]["count"] + 1 .....(increase epc count by one
      
      -   Check  for the 'sub_ref_no' is_submit is true or false
          - if it is true then scanning is stopped warning
      -   Iteratting an `raw_scan_metadata`for calculate total of scanned_epc in variable `total_found_count`
      -   Update all the comparision values in the database table
   
                  {# Raw Scan
                    "epc": raw_scan_epcs_,
                    "barcode": raw_scan_barcodes_,
                    "raw_scan_metadata": raw_scan_metadata_,
                    "total_found_count": len(raw_scan_epcs_)
                   }
          
      -    Send raw_scan_data to the UI

                           { 
                             "total_found_count": len(raw_scan_epcs_),
                             "raw_scan_metadata": raw_scan_metadata_,
                             "expected_count":None,
                             "valid_found_count":None,
                             "extra_found_count":None
                            }



# Inwarding
## **Step 1**

1] APi for the fetch all invoices from cloud

`@router.get('/tunnel/inwarding/get_invoices', tags=["StyleUnion/Singular In-warding Tunnel"])`

2] Api for fetch all boxes of specific invoice from cloud

`@router.get('/tunnel/inwarding/get_boxes', tags=["StyleUnion/Singular In-warding Tunnel"])`

3] Api for the fetch box_data of specific box_number from cloud

`@router.get('/tunnel/inwarding/get_box_data', tags=["StyleUnion/Singular In-warding Tunnel"])`

## **Step 2**

1] Firstly call the websocket tunnel_api from UI

`@router.websocket("/ws/tunnel-read/")`

2] Connect to websocket using `WebSocket` and accept the connection.

3] accept the text from user using `WebSocket` in variable `soc_data`.

4] Ui sends status in `soc_data` as "NEW_CONNECTION"

   - Check panel is not deleted in database.
     - check the sensor details for the process.
     - Check `is_sensor_enabled` is False
        - then `sensor_flag set to "False".
     - else
       - get reader data from db and store in variable `sensor_control_data`

   - Set `sub_ref_no` from soc_data["data"]

   - After that send json `{
                    "status": "INSERT_BOX",
                    "data": []
                }`  to the UI for the inserting a box inside the Tunnel..

   - In Singular methods release thread `singular_entrypoint` passing parameters is `(sub_ref_no, sensor_control_data, sensor_flag, soc_data, db)`
   - In `singular_entrypoint`  run `start_singular_polling_data` passing parameters `(sub_ref_no, sensor_process_data, sensor_flag, soc_data, db)`
   - In `start_singular_polling_data` function
     - method will be "inwarding". It is taken from UI json like `soc_data['method']`.
     - fetching metadata_id from database, which is new entry is created after getting metadata of the specific box.
     - Initially `match_flag` set to False.
     - `process_stages` added in db like `(Start Scanning,Stop Scanning,Data Matched,Data UnMatched)` with `process_stage_code`.
        - In `sm_data` json stores process_data as "process_stage_code" as key and all "data of process" as value.
           - ![Screenshot from 2023-11-17 15-52-50](https://github.com/Aishwarya1307/Doccumetation/assets/125255809/10228819-1abc-436a-aee4-45dbeaa1c714)

      - In while loop
        - Check start_scanning[method] is True
          - input_box_state is set to True
        - else
          - check sensor_flag is True
            - set input_box_state from `read_gpio_for_input`
        - check `input_box_state` is True
           -  When box is inside the Tunnel then send json as response to the UI as `{
                    "status": "BOX_INSIDE_TUNNEL",
                    "data": []
                }`
           -  send_gpio_trig_zone("OUT4")
         - Into Inner While loop
            - Firstly check `redis_connection` not exists 
              -  calling :   def update_metadat_for_scanning_stopped(sub_ref_no, soc_data, db)  For the scanning status is update in db as `stopped` according to `sub_ref_no`.
              -  Send json to the UI for check the reader connection.
            -  declare `scannned_epc` list is empty.
            -  getting data of epcs from redis db which is stored in key method name and epc
               - ![Screenshot from 2023-11-17 16-10-17](https://github.com/Aishwarya1307/Doccumetation/assets/125255809/b2dfa799-9b77-47b6-882a-53aa8e8a6d2d)
               - `data` is a list of "epcs".

            - Check redis `data` is greater than 0
              - set `barcode_data` is empty dictionary
              - fetch one by one epc from data
                - Check epc_ is starts with `B1` , If it is True then ignore that epc 
                -check epc not in `scanned_epc` from line `173`
                   - append that epc in `scanned_epc`
                - decode the epc in the `sgtin96_decode(epc_)` function
                -  fuction responce if epc is decodable then responce like `{"product_code": barc, "serial": pyepc.decode(epc).serial_number}` , Otherwise epc is does not decode or not valid then responce like `{"product_code": "NOT_IN_SOH", "serial": 200}`
                - Response stored in the `barcode_` variable.
                - Check `barcode_['product_code']` is in barcode_data.keys()
                  - append the `epc_` in barcode_data[barcode_['product_code']
                - else
                  - add new key value in the dictionary as `barcode_data[barcode_['product_code']] = [epc_]`
                - ![Screenshot from 2023-11-17 16-31-35](https://github.com/Aishwarya1307/Doccumetation/assets/125255809/82ffa6c0-4ec8-4580-ab19-31a2f2290f42)
             
                - **Compare_epc_logic**
                   - passing parameters to the `compare_epc_logic` is
                     - `method` ===: which type of method is scanning
                     - `sub_ref_no` ===: box_number
                     - `barcode_data` ===: json of scanned epcs like {"product_code":[epc1,epc2]}
                     - `metadata_id` ===: entry of box number id from db
                  - Check which type of method
                    - method =='Inwarding'
                      - get data from db respected to the metadata_id in veriable `com_met_data`.
                      - calling **process_inwarding_data** functon
                        - passing parameters to the function 
                          1. `method` ===:whitch type of method
                          2. `sub_ref_no` ===: box_number
                          3. `com_met_data` ===: data of present box number which is store in db filter with `metadata_id`
                          4. `barcode_data` ===: json of scanned epcs like {"product_code":[epc1,epc2]}
                          5. `match_flag` ===: data is matched to the scanned data
                        - Firstly check `com_met_data` is null
                          1. raise Exception as "Scan_id is None"
                        - Scanned data whitch is stored in db that is initialize as below:
                            ![Screenshot from 2023-11-17 17-45-55](https://github.com/Aishwarya1307/Doccumetation/assets/125255809/12f4b1f0-6457-4735-8d3a-13a8c470fb86)

                        - getting scan_barcode from barcode_data.keys()
                          1. Check scan_barcode in expected_barcodes which is from db
                          2. 
                    
                  
                    













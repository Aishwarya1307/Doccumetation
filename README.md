# Raw Scan

**Step 1** 

   
-   When box insert into the Tunnel then , scannig process starts and scanned tags are added in redis_database.
-   https://github.com/iam-rnd/tagid-iot-gateway/blob/82f5f3b3aa0223397730e94649304a2ee3e2062a/src/routes/SocketManager/CrimsouneClub/socket_manager.py#L72
-   
     ![Screenshot from 2023-11-02 17-16-24](https://github.com/Aishwarya1307/CREATE_REACT_APP/assets/125255809/61ebc93d-d5de-44eb-a952-9f59823ec2ab)

-   When box insert into the Tunnel then send json to UI `{
                    "status": "BOX_INSIDE_TUNNEL",
                    "data": []
                }` 

-   Getting scanned tags data from redis as well as maintaining unique epc's.
-   The `sgtin96_decode` function,decodes the scanned epc's.
-   In `barcode_data` variable, stores a json '{"barcode1":[epc1,epc2,epc3] ,"barcode2":[epc4,epc5,epc6]}' with unique epc's and barcodes.
-   https://github.com/iam-rnd/tagid-iot-gateway/blob/82f5f3b3aa0223397730e94649304a2ee3e2062a/src/routes/SocketManager/CrimsouneClub/socket_manager.py#L103
   
      ![Screenshot from 2023-11-02 17-41-32](https://github.com/Aishwarya1307/CREATE_REACT_APP/assets/125255809/587ba34a-c844-43af-901e-0f6f191e3a2c)

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

  ![Screenshot from 2023-11-02 18-25-32](https://github.com/Aishwarya1307/CREATE_REACT_APP/assets/125255809/4d767142-7efc-459d-b3c7-1e004e45abbf)


    

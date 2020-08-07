# telemedicine-Project
Im working on a small project and im in a bind
pragma solidity ^0.4.26;

import "github.com/ethereum/dapp-bin/library/stringUtils.sol";

contract ElectronicHealthRecords {
    bool public status;
    address public Admin;
    address public EHRsManager;
    uint public numberOfUsers;
    mapping (address=>uint) public userId;
    User[] public users;
    
    event UserAdded(address UserPK, string UserRole);
    event UserRemoved(address UserPK);
    event DataResult(address UserPK);
    event ReturnAccessResult(address index_from, string message, bool result, uint time, uint penalty);
    
    //Creating a data structure for storing variables of different data types
    struct User {
        address user;// from the original code "address userPK"
        string role;// from the original code "address userRole"
    }
    
    struct EHRstorage {
        string storageName;
        address AreaID;
        string Hash_value;
        address PatientID;
    }
    
    struct PolicyItem { //for <EHRs cloud resource, action>; {
        bool userPK;
        bool AreaID;
        bool PatientID;
        string permission; //verify permission: "allow" or "deny"
        bool result; //ReturnAccessResult
        bool checkID; //check the access
    }
    
    mapping (bytes32=>EHRstorage)
    public DTHtable;
    mapping (bytes32=>mapping(bytes32=>PolicyItem)) policies; ///mapping(EHRs resource, action) to policy checking
    
    modifier onlyAdmin {
        require(msg.sender==Admin);
        _;
    }
    
    function SC_creation()public {
        Admin=msg.sender;
        status=true;
        User(Admin, 'Creator of Smart Contract');
        numberOfUsers=0;
    }
    
    function addUser(address userPK, string userRole)
    onlyAdmin public {
        require(status = true);
        uint id = userId[userPK];
        if(id==0) {
            userId[userPK]=users.length;
            id=users.length++;
        }
        users[id]= User({user:userPK, role:userRole});
        emit UserAdded(userPK, userRole);
        numberOfUsers++;
    }
    
    function removeUser(address userPK){
        onlyAdmin; {
            require(userId[userPK]!=0);
            for (uint i=userId[userPK]; i<users.length-1;i++){
                users[i]=users[i+1];
            }
            delete users[users.length-1];
            users.length--;
            emit UserRemoved(userPK);
            numberOfUsers--;
        }
    }
    
        function stringToBytes32(string memory source) returns (bytes32 result){
           bytes memory tempEmptyStringTest = bytes(source);
             if (tempEmptyStringTest.length == 0) {
                return 0x0;
            }
            assembly {
                result := mload(add(source, 32))
            }
        }
        function stringToBytes32Memory(string memory source) returns (bytes result )
        {
           bytes memory b3 = bytes(source);
           return b3;
        }
        
        function retrieveEHRs(string stringToBytes32, string retrieve_name, address userPK,string storageName, address AreaID, address PatientID, string Hash_value)
        onlyAdminp public{
            bytes32 key=stringToBytes32(retrieve_name);
            /'Look up the EHRs data in DHT table'/
            DTHtable[key].storageName=storageName;
            DTHtable[key].AreaID=AreaID;
            DTHtable[key].PatientID=PatientID;
            DTHtable[key].Hash_value=Hash_value;
            emit DataResult(userPK);
            //DTHtable[key];
        }
        
        function policyList(address EHRsManager,stringToBytes32,'/ string_resource, string_action, string_permission /'address userPK, address AreaID, address PatientID'/)
        public{
            bytes32 EHRresource=stringToBytes32(_resource);
            bytes memory action=stringToBytesMemory(_action);
            if (msg.sender==EHRsManager){
                policies[EHRresource][action].userPK=true;
                policies[EHRresource][action]..AreaID=true;
                policies[EHRresource][action].PatientID=true;
                policies[EHRresource][action].permission=_permission;
                policies[EHRresource][action].result=false;
            }
            else revert();
        }
        
        function penalty( string_resource, string_action, uint time )
        public{
            //bool policycheck=false;
            //bool behaviorcheck=true;
            bool userPKcheck = false;
            bool AreaIDcheck = false;
            bool PatientIDcheck = false;
            bool checkID = flase;
            uint penalty=0;
            bytes32 EHRresource_Conv=stringToBytes32(_resource);
            bytes memory action=stringToBytesMemory(_action);
            if (msg.sender==EHRsManager){
                userPKcheck=policies[EHRsresource_Conv][action].userPK;
                userPKcheck=policies[EHRsresource_Conv][action].AreaID;
                PatientIDcheck=policies[EHRsresource_Conv][action].PatientID;
                checkID=userPKcheck && AreaIDcheck && PatientIDcheck;
                if(checkID==true) //Detect authorized access
                emit ReturnAccessResult(msg.sender, "Successful!", true, time, Penalty);
                else //Detect an unauthorized access
                emit ReturnAccessResult(msg.sender, "Failed!", true, time, Penalty);
            }
        }
    }

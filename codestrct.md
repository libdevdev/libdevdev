/*
* Code guide line for libdvdv
*/

/libdvdvign
    -> BuildIgnoreFile( );
        # Builds an ignore file in project directory
        # if a .gitignore file is present, it copies the input form it.

    -> ParseIgnoreFile( );
        # parses the ignore file, and pass it to build ignore list.

    -> BuildIgnoreList( );
        # this sets up the list of files that need to be ignored
        # from this project.

    -> Check( )
        # returns a boolean, weather a file should be ignored or not.

/libdvdvupdate
    -> Setup( );
        #initializes state database with initial reading.

    -> BuildUpdate( ) 
        # this function takes the current project state
        # and builds the tar.gz file containing the update.
            
    -> ApplysUpdate( tar.gz )
        # applies the update patch contained inside of 
        # tar.gz file.

/libdvdvstorage
    -> Store( file ):
       # stores files to file server.

    -> Get( url ):
        #gets file from file server.


/libdvdvserver
    -> http post /export?surl=...&f0=...&f1=...&f2=...&fn=...&UUID0=...UUIDn=....:
            # f0...fn are module urls stored in storage...
            # UUID...UUIDn are uniquely genrated uuids.
            # surl is project tar.gz url
            sid := new sid;
            usrid := cookie.GetUserId();
            Sql.AddToWork(surl, sid, usrid);
            Sql.AddToTableWorkModules((sid,f,UUID0), (sid, f1, UUID1), ....(sid, fn, UUIDn));      
            return sid;

    -> http post update /update?surl=...uuid0=...&uuid1=...&uuidn=...:
        


/libdvdvwallet
    -> CreateWallet():

    -> GetBalance():
        if libdvdvutil.PathExist("~/.libdvdvwallet"):
            wallet := decrypt("~/.libdvdvwallet"); //asks for the password.
            return getBalance(wallet.pubkey);
        else :
            return -1;           
             

main:
    if "libdvdv m init":
        os.Mkdir(project_path + ".libdvdv") #create the libdvdv directory
        libdvdvign.BuildIgnoreFile(); //this builds the ignore file.

    if "libdvdv m allocwrk":
        BuildIgnoreList(ParseIgnoreFile());
        modules := SearchModules();
        uuids := libdvdvutil.GenUuids(len(modules));
        price := GetPrice(modules);
        if libdvdvwallet.GetBalance() >= price:
            url := libdvdvstoreage.Store(libdvdvutil.tar(".")); 
            urlf := libdvdvstoreage.Store(modules); 
            sid := libdvdvserver.PostExport('surl={url}&f0={urlf0}....&fn={urlfn}&UUID0={uuids0}...UUIDn={uuidn});
            os.WriteFile(".libdvdv/uuids.i", uuids);
            os.WriteFile(".libdvdv/sid.i", sid);
            libdvdvupdate.Setup();
        else:
            return;

    if "libdvdv m pushupdate":
        BuildIgnoreList(parseIgnoreFile());
        tar.gz := BuildUpdate(".");
        url := libdvdvstorage.Store(tar.gz);
        sid := os.ReadFile(".libdvdv/sid.i");
        libdvdvserver.PostUpdate(url, sid);
    
    if "libdvdv m wallet":
        
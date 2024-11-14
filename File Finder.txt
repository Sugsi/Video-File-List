# Define the drive to search
$drive = "D:\"  # Change to the drive you want to search

# Set the output directory
$outputDir = "D:\Video File Lists"

# Check if the output directory exists, and create it if it does not
if (!(Test-Path -Path $outputDir)) {
    New-Item -ItemType Directory -Path $outputDir | Out-Null
}

# Define the base filename
$baseFileName = "video_files_list"
$outputFile = "$outputDir\$baseFileName.txt"

# Check if the output file already exists, and increment the filename if necessary
$counter = 1
while (Test-Path -Path $outputFile) {
    $outputFile = "$outputDir\${baseFileName}_$counter.txt"
    $counter++
}

# Define video file extensions to look for, including .ts
$videoExtensions = @("*.mp4", "*.avi", "*.mkv", "*.mov", "*.wmv", "*.flv", "*.mpeg", "*.m4v", "*.ts")

# Find all video files on the drive and sort by size (largest to smallest)
$videoFiles = Get-ChildItem -Path $drive -Recurse -Include $videoExtensions -ErrorAction SilentlyContinue |
              Sort-Object -Property Length -Descending

# Get the total used space on the drive (D:) 
$driveInfo = Get-PSDrive -Name 'D'
$usedSpaceGB = [math]::round(($driveInfo.Used / 1GB), 2)
$usedSpaceTB = [math]::round(($driveInfo.Used / 1TB), 2)

# Determine the used space in GB or TB based on the size
$usedSpace = if ($usedSpaceTB -gt 1) {
    "$usedSpaceTB TB used"
} else {
    "$usedSpaceGB GB used"
}

# Write the title with used space at the top
"        In Order Of Biggest To Smallest - $usedSpace`r`n" | Out-File -FilePath $outputFile -Encoding UTF8

# Write the file names with indentation and page breaks to the output file
$videoFiles | ForEach-Object {
    $fileName = $_.Name
    $fileSize = $_.Length

    # Conditionally set the file size unit
    if ($fileSize -ge 1GB) {
        $fileSizeFormatted = [math]::round($fileSize / 1GB, 2)
        $sizeUnit = "GB"
    } elseif ($fileSize -ge 1MB) {
        $fileSizeFormatted = [math]::round($fileSize / 1MB, 2)
        $sizeUnit = "MB"
    } else {
        $fileSizeFormatted = [math]::round($fileSize / 1KB, 2)
        $sizeUnit = "KB"
    }

    "`r`n        $fileName - $fileSizeFormatted $sizeUnit"  # Indented with 8 spaces and page break before each entry
} | Out-File -FilePath $outputFile -Encoding UTF8 -Append

# Display completion message
Write-Output "Video file list created at $outputFile"

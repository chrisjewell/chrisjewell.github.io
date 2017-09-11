
## Payroll File Converter for Esperanza Health Center

Every two weeks, I receive a big Excel spreadsheet from our payroll company that contains all of the payroll data for that particular pay period. My job is to upload a subset of those data onto our 401(k) recordkeeper's web portal in order to make sure that everyone's 401(k) contributions get deposited and invested. In the past, I've done this by performing a series of manual edits to a copy of the spreadsheet, and then feed the remaining data into a conversion template that I built in order to get the information in the format that the 401(k) recordkeeper wants to see. This was a simple but tedious task, and after a little while, I sought to find ways to make the process more automatic. This is where Python comes in.

I recently wrote the following script, which takes the big spreadsheet I receive, and performs all of the edits that I need, and then spits out the edited files on the other end. This will be a bit of a time-saver, making a 10-15 minute task into a 3-5 second one. Not bad!

Feel free to reach out and let me know if you have any questions or feedback - I'd love to hear from you!

This first section imports all necessary modules.

GOALS: I would like this to be able to take the file from the downloads folder; [DONE]
make all the necessary copies for archival/work puposes; [DONE]
edit, save [DONE] and email the FSA Upload;
edit and save data into the Contibutions upload file [DONE]


```python
import openpyxl as o
from openpyxl.utils.dataframe import dataframe_to_rows
import glob, os, shutil
import numpy as np
import pandas as pd
import itertools as it
#import datetime as dt
#import webbrowser as web
```

This next section of code defines the necessary variables for this script. As a note, it is set to downloads by default now, but 


```python
#This is the source of the file from our payroll company - Downloads by default
srcfolder = 'C:/Users/cjewell/Downloads'

#This is where you will save closed system archive files (historical files that are less easily accessed by the user)
contrib_archive = 'files/archive_contribution' #C:/Users/cjewell/Desktop/DataProjects/pension/
fsa_archive = 'files/archive_fsa' #C:/Users/cjewell/Desktop/DataProjects/pension/

#This is where you will save the active/working files to load into openpyxl - This is a CLOSED folder, and files will not be saved
sys_contrib_active = 'files/sys_contrib_active' #C:/Users/cjewell/Desktop/DataProjects/pension/
sys_fsa_active = 'files/sys_fsa_active' #C:/Users/cjewell/Desktop/DataProjects/pension/

#This is where you will save open archive files (historical files that can be easily accessed by the user)
contrib_open_archive = 'files/Contributions' #C:/Users/cjewell/Desktop/DataProjects/pension/
FSA_open_archive = 'files/FSA Upload' #C:/Users/cjewell/Desktop/DataProjects/pension/
```

This section defines all of the functions for this script.


```python
def copy(src, dest):
    shutil.copy2(src, dest)

def copy_contrib(src,dest):
	#Takes to dir paths as arguments, and copies the Health Benefit Pension Reports to the correct record folders.
    for filename in glob.iglob(os.path.join(src,'*Health Benefits-Pension Report.xlsx')):
        new_file = str((os.path.join(dest,('20' + filename[39:41] + ' ' + filename[33:35] + ' ' + filename[36:38] + '.xlsx'))))
        copy(filename, new_file)

def copy_FSA(src,dest):
	#Takes to dir paths as arguments, and copies the Health Benefit Pension Reports to the correct record folders.
    for filename in glob.iglob(os.path.join(src,'*Health Benefits-Pension Report.xlsx')):
        new_file = str((os.path.join(dest,('FSA ' + '20' + filename[39:41] + ' ' + filename[33:35] + ' ' + filename[36:38] + '.xlsx'))))
        copy(filename, new_file)

def delete_src_file(src):
    # Removes original file from source folder
    for filename in glob.iglob(os.path.join(src,'*Health Benefits-Pension Report.xlsx')):
        os.remove(filename)

def delete_test_file(src):
    # Removes a particular test file
    for filename in glob.iglob(os.path.join(src,'*.xlsx')):
        os.remove(filename)
        
def delete_test_results():
    # Removes test files from all folders used by this script
    delete_test_file(contrib_archive)
    delete_test_file(fsa_archive)
    delete_test_file(sys_contrib_active)
    delete_test_file(sys_fsa_active)
    delete_test_file(contrib_open_archive)
    delete_test_file(FSA_open_archive)
    print 'Test files cleared.'
    
def convert_to_positive(ws, col):
    # Takes an openpyxl worksheet and an excel column reference (e.g. 'A') as arguments
    # Skips cells that contain string values as well as None values, and multiplies all float values by -1 to convert to positive
    for cell in ws[col]:
        if cell.value != None:
            if isinstance(cell.value, (str, unicode)):
                pass
            else:
                cell.value = cell.value * -1
        else:
            pass
```

This section copies the file from "Downloads" into the archive folders and deletes the original.


```python
# Delete previous test copies
delete_test_results()

# Make active working copies of original file
copy_contrib(srcfolder,sys_contrib_active)
copy_FSA(srcfolder,sys_fsa_active)

# Make system archive copies of the original file
copy_contrib(srcfolder,contrib_archive)
copy_FSA(srcfolder,fsa_archive)

# Delete original file from srcfolder
#delete_src_file(srcfolder)

# Import Personnel file to join onto payroll file for employee IDs
personnel = pd.read_csv('files/Personnel.csv', index_col='Payroll ID') #
```

    Test files cleared.
    

This section begins the editing process with the FSA Upload files.

This section takes the worksheet and moves it into Pandas for cleaning and editing


```python
for filename in glob.iglob(os.path.join(sys_fsa_active,'*.xlsx')):
    #Port the sheet to Pandas for editing
    wb = o.load_workbook(filename)
    ws = wb.active
    data = ws.values
    cols = next(data)[1:]
    data = list(data)
    idx = [r[0] for r in data]
    data = (it.islice(r, 1, None) for r in data)
    df = pd.DataFrame(data, index=idx, columns=cols)
    
    
    # Edit the sheet/dataframe in Pandas
    cols = df.columns.values.tolist()

    df = df.drop(cols[1:13], axis=1)
    df = df.drop(cols[14:19], axis=1)
    df = df.drop(cols[20:22], axis=1)
    df = df.drop(cols[29:], axis=1)
    
    # Port the dataframe back to openpyxl and save the worksheet as .xlsx file
    wb1 = o.Workbook()
    ws1 = wb1.active

    for r in o.utils.dataframe.dataframe_to_rows(df, index=False, header=True):
        ws1.append(r)

    wb1.save(filename)
    
# Make finished copies of the edited files

for filename in glob.iglob(os.path.join(sys_fsa_active,'*.xlsx')):
    copy(filename, FSA_open_archive)
    os.remove(filename)

print 'FSA files completed.'
```

    FSA files completed.
    

This section begins the editing process with the Contribution files.

This section takes the worksheet and moves it into Pandas for cleaning and editing


```python
# Set neg_cols variable with all of the column references that have negative values in them.
neg_cols = ['E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V']


for filename in glob.iglob(os.path.join(sys_contrib_active,'*.xlsx')):
    # Port the sheet to Pandas for editing
    wb = o.load_workbook(filename)
    ws = wb.active
    data = ws.values
    cols = next(data)[1:]
    data = list(data)
    idx = [r[0] for r in data]
    data = (it.islice(r, 1, None) for r in data)
    df = pd.DataFrame(data, index=idx, columns=cols) #
    
    
    # Edit the sheet/dataframe in Pandas
    cols = df.columns.values.tolist()

    # Drop unnecessary columns
    df = df.drop(cols[0:2], axis=1)
    df = df.drop(cols[3:6], axis=1)
    df = df.drop(cols[7:13], axis=1)
    
    # Drop blank/duplicate rows
    df = df.dropna(axis=0, subset=['Date','Gross'], how='any') # Drops both the duplicate rows and the summary rows with totals
   
    # Join on Quickbase Employee ID# from Personnel.csv
    df = df.join(personnel['Employee ID#'], how='left') # on=personnel['Payroll ID'],
    
    # Rename column headers
    df.rename(columns={'K401': 'Employee 401(k)', 'K401_ER': 'Match', 'A401K_ERa': 'Profit Sharing'}, inplace=True)
    
    # Open new openpyxl workbook, and port the dataframe back to openpyxl
    wb1 = o.Workbook()
    ws1 = wb1.active

    for r in o.utils.dataframe.dataframe_to_rows(df, index=True, header=True):
        ws1.append(r)    

    # Convert Date format to MM/DD/YYYY
    date = o.styles.named_styles.NamedStyle(name='datetime', number_format='MM/DD/YYYY')
    for cell in ws1['B']:
        cell.style = date
        
    # Convert negative numbers to positive
    for column in neg_cols:
        convert_to_positive(ws1, column)
        
    #Save the workbook as .xlsx file
    wb1.save(filename)
    
# Make finished copies of the edited file

for filename in glob.iglob(os.path.join(sys_contrib_active,'*.xlsx')):
    copy(filename, contrib_open_archive)
    os.remove(filename)

print 'Contribtution files completed.'
```

    Contribtution files completed.
    


```python
print 'All files completed.'
```

    All files completed.
    

// Index.html
{/* <div>
<h1>File upload</h1>
<input type="file" multiple onChange={e => uploadFile(e)}/>
</div> */}


// Index.js
// const uploadFile = async e => {
// const files = e.target.files
// console.log('files', files)
// const form = new FormData()
// for (let i = 0; i < files.length; i++) {
// form.append('files', files[i], files[i].name)
// }
// try {
// let request = await fetch('/upload', {
//   method: 'post',
//   body: form,
// })
// const response = await request.json()
// console.log('Response', response)
// } catch (err) {
// alert('Error uploading the files')
// console.log('Error uploading the files', err)
// }
// }



// Server.js

const bluebird = require('bluebird')
// const fs = bluebird.promisifyAll(require('fs'))
const Formidable = require("formidable")
const { join } = require("path")
const fs = require('fs');
const { Console } = require('console');
const fsPromises = require('fs').promises;

// Returns true if successful or false otherwise
async function checkCreateUploadsFolder(uploadsFolder) {
    console.log("uploadsFolder", uploadsFolder)
    try {
        // check this folder has or not
        await fsPromises.stat(uploadsFolder);
        console.log('Directory already exists.');
    } catch (e) {
        // if folder not exist error.code : 'ENOENT' 
        if (e && e.code == 'ENOENT') {
            console.log('The uploads folder doesn\'t exist, creating a new one...')
            try {
                // if file not exist make this folder
                await fsPromises.mkdir(uploadsFolder);
                console.log('Directory created successfully.');
            } catch (mkdirErr) {
                console.error('Error creating directory:', mkdirErr);
            }
        } else {
            console.log('Error creating the uploads folder 2')
            return false
        }
    }
    return true
}

// Returns true or false depending on whether the file is an accepted type
function checkAcceptedExtensions(file) {
    const type = file.mimetype.split('/').pop()
    const accepted = ['jpeg', 'jpg', 'png']
    if (accepted.indexOf(type) == -1) {
        return false
    }
    return true
}

upload = async (req, res) => {
    // receive file and fild for form
    let form = new Formidable.IncomingForm()
    console.log("------------form----------")

    // make upload directory getCurrentFolderPath = __dirname than ../../ go main folder
    const uploadsFolder = join(__dirname, '../../../public', 'uploads')
    // if wanted multiples file to make true otherwise single file receive
    form.multiples = true
    // where file upload know that
    form.uploadDir = uploadsFolder
    //inisilize file size if we want
    form.maxFileSize = 50 * 1024 * 1024 // 50 MB
    // check folder if or not
    const folderCreationResult = await checkCreateUploadsFolder(uploadsFolder)
    if (!folderCreationResult) {
        return res.json({ ok: false, msg: "The uploads folder couldn't be created" })
    }
    form.parse(req, async (err, fields, files) => {
        let name = ''
        for (const fieldName in files) {
            name = fieldName
        }
        // if error file come
        if (err) {
            console.log('Error parsing the incoming form')
            return res.json({ ok: false, msg: 'Error passing the incoming form' })
        }
        // Check if files were sent
        if (!files || Object.keys(files).length === 0) {
            console.log("No files received or files object is empty");
            return res.status(400).json({ ok: false, msg: 'No files were submitted with the form' });
        }

        console.log("files", files[name].length)
        // If we are sending one by one file:
        for (let i = 0; i < files[name].length; i++) {
            // one file distracture
            const file = files[name][i]
            // check file type accepted or not
            if (!checkAcceptedExtensions(file)) {
                console.log('The received file is not a valid type')
                return res.json({ ok: false, msg: 'The sent file is not a valid type' })
            }
            // make file name use date 
            const currentDate = new Date().toISOString().replace(/[-T:Z.]/g, '');
            const fileName = currentDate + '_' + encodeURIComponent(file.originalFilename.replace(/&.*;+/g, '-'));
            try {
                // save file ===file path == where upload == file name
                await fsPromises.rename(file.filepath, join(uploadsFolder, fileName))
            } catch (e) {
                console.log('Error uploading the file')
                // if fail save than remove file that save 
                try { await fsPromises.unlink(file.filepath) } catch (e) { }
                return res.json({ ok: false, msg: 'Error uploading the file mul' })
            }
        }
        return res.json({ ok: true, msg: 'Files uploaded succesfully!', })
    })
}
module.exports = {
    upload,
};
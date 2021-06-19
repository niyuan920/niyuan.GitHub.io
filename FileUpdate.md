前台页面

      <form method="post" enctype="multipart/form-data" 
          asp-action="AddFile"
         asp-controller="Patent">
        <input type="file" id="addFile" name="file" multiple>
        <input type="hidden" name="formNo" id="number"/>
        <input class="btn btn-outline-primary" type="submit" value="上传" onclick=" getFile();"/>
    </form>
    
Controller文件上传  

    public IActionResult AddFile(IFormFile file, string formNo)
        {
            var dto = new ResultDTO();
            var entiy = new File();
            string[] type = file.FileName.Split(".");
            entiy.FileName = file.FileName;
            entiy.FormNo = int.Parse(formNo);
            entiy.Size = ConvertFileSize(file.Length);
            entiy.Format =type[type.Length-1]; 
            entiy.CreateDate = DateTime.Now;
            var success = UploadFile(file,entiy);
            if (success)
            {                            
                var res = PatentBiz.AddFile(entiy);
                if (res)
                {
                    dto.Status = true;
                    dto.Message = "上传成功！";
                }
                else
                {
                    dto.Status = false;
                    dto.Message = "文件上传失败";
                }
            }
            else
            {
                dto.Status = false;
                dto.Message = "文件上传失败";
            }
            dto.FormNo = int.Parse(formNo);
            return RedirectToAction("FileList",dto);
        }
        public bool UploadFile(IFormFile file,File entiy)
        {
            if (file != null)
            {

                var fileDir = @"F:\IDEA\IDEA\IDEA.Repository\UploadFiles";


                if (!Directory.Exists(fileDir))
                {
                    Directory.CreateDirectory(fileDir);
                }
                //文件名称
                string guid = Guid.NewGuid().ToString();
                string projectFileName = guid + "_"+file.FileName;
                entiy.Site = projectFileName;
                entiy.FileId = guid;
                //上传的文件的路径
                string filePath = fileDir + $@"\{projectFileName}";
                using (FileStream fs = System.IO.File.Create(filePath))
                {
                    file.CopyTo(fs);
                    fs.Flush();
                }
                return true;
            }
            else
            {
                return false;
            }
        }

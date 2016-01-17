---
layout: post
title: 模拟form表单上传图片
description: JS模拟form表单上传图片，Node服务器接收图片
category: fe
---

浏览器端：
html：

	 <form id="fileupload" class="fileupload form-upload" style="display:none;" enctype="multipart/form-data" method="post" action="javascript:;" role = "form">\
		<input type = "file" name = "files" class="file-input" /> \
		<input type="hidden" name="modulename" value="{{modulename}}"/>\
		<input type="button" class="upload-btn J_Uploadimg"  value="上传截图"/>\
	</form>\
			 
JS：


	var $fileUpload = $(".fileupload");

	var formData = new FormData($fileUpload[0]);

		$.ajax({
			url: '/api/upload_img',
			type: 'POST',	
			data: formData,
			async: false,
			cache: false,
			contentType: false,
			processData: false,
			success: function(data){
				nanobar.go(100);
				//异步化
				if(data.success){
					$('#J_MsgCenter').html($.substitute('<span class="label label-success">{message}</span>',{message:data.message}));
				}
				else{
					$('#J_MsgCenter').html($.substitute('<span class="label label-danger">{message}</span>',{message:data.message}));
				}	
				node.removeClass('disabled');
				if(node.attr('data-reload')){
					window.location.reload();
				}
				console.log('imgUploader upload success, data:', data);
			},
			error: function(){
				$("#spanMessage").html("与服务器通信发生错误");
			}
		});
		
服务器端：

	module.exports = function (req, res, next) {
    
    new Promise(function(resolve, reject){

    res.append('Access-Control-Allow-Origin', '*');

    var form = new multiparty.Form();

    form.parse(req, function(err, fields, files) {

      console.log('fields:');
      console.log(fields);

      console.log('files:');
      console.log(files);

      var modulename = fields.modulename[0];

      var modulePath = path.join(work_path, 'module', modulename);
      var imgPath = files.files[0].path;

      uploadImg(imgPath, modulePath, resolve, reject);

    });

  })


[Joebon]:    http://joebon.tk  "Joebon"

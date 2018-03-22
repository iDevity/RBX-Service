Welcome to our site, please don't worry, this website is currently in works.

<button type="button">Discord</button>



require "json"
require "net/http"
require 'net/https'
require "uri"

def sendData(url, data)
	uri = URI.parse(url)
	req = Net::HTTP::Post.new(uri, 'Content-Type' => 'application/json')
	req.body = data
	res = Net::HTTP.start(uri.hostname, uri.port, :use_ssl => true, :verify_mode => OpenSSL::SSL::VERIFY_NONE) do |http|
		http.request(req)
	end
	return res
end

def setStatus(res)
	case res.code.to_i
		when 200
			@status.text = "Sent!"
		when 301
			@status.text = "The resource has moved."
		when 400
			@status.text = "Error: Bad Request"
			_data = JSON.parse(res.body)
			_statusText = "";
			if _data.has_key? 'icon_url'
				_statusText += "\nIcon URL Error\n#{_data["icon_url"][0]}"
			end
			if _data.has_key? 'attachments'
				_dt = _data['attachments'][0]
				if _dt.include? 'color'
					_statusText += "\nInvalid Color Value"
				end
				if _dt.include? 'footer_icon'
					_statusText += "\nInvalid Footer Icon Value"
				end
				if _dt.include? 'author_icon'
					_statusText += "\nInvalid Author Icon Value"
				end
				if _dt.include? 'author_link'
					_statusText += "\nInvalid Author Link Value"
				end
			end
			if _data.has_key? 'message'
				_statusText += "\n#{_data['message']}"
			end
			@status.text = _statusText
		when 401
			@status.text = "Error: Unauthorized"
		when 404
			@status.text = "The resource wasn't found."
		when 405
			@status.text = "Error: The method isn't allowed."
		when 429
			@status.text = "Ratelimited. Try again later."
		else
			@status.text = "Error while sending request: #{res.code}."
	end
end

Shoes.app width: 400, height: 600, resizable: false, title: "Webhook Sender" do
	stack do
		flow do
			@url = edit_line "Webhook URL"
			@url.style(width: 0.5)
			@send = button "Send" do
				@data = {
					"username" => @username.text(),
					"icon_url" => @icon_url.text(),
					"text" => @text.text(),
					"attachments" => [
						{
							"color" => @color.text(),
							"title" => @title.text(),
							"pretext" => @pretext.text(),
							"author_name" => @author_name.text(),
							"author_link" => @author_link.text(),
							"author_icon" => @author_icon.text(),
							"footer" => @footer.text(),
							"footer_icon" => @footer_icon.text(),
							"text" => @attachment_text.text()
						}
					]
				}

				@status.text = "Sending"
				@k = sendData(@url.text(), @data.to_json)

				setStatus(@k)
			end
			@send.style(width: 0.5)
		end
		flow do
			@metaStack = stack do
				@username = edit_line "Username"
				@icon_url = edit_line "Icon URL"
			end
			@text = edit_box "Text"
			@metaStack.style(width: 0.5)
			@text.style(width: 0.5, height: 56)
		end
		title "Attachments", align: "center"
		flow do
			@attachmentStack = stack do
				@color = edit_line "Color"
				@title = edit_line "Title"
				@pretext = edit_line "Pretext"
				@author_name = edit_line "Author Name"
				@author_link = edit_line "Author Link"
				@author_icon = edit_line "Author Icon"
				@footer = edit_line "Footer"
				@footer_icon = edit_line "Footer Icon"
			end	
			@attachment_text = edit_box "Text"
			@attachmentStack.style(width: 0.5)
			@attachment_text.style(width: 0.5, height: 224)
		end

		@status = para "Waiting..."
		@status.style(align: "center", margin_top: 25)

		flow do
			@licence = para("Made by ", l = link('Mackan', click: "http://github.com/Sven65"))
			@licence.style(bottom: 5, margin_top: 150, margin_right: 5, align: "right")
		end

	end
end

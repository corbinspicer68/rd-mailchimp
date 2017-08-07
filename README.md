# Mailchimp

Вам необходимо взять исходники из файла репозитория js/core.min.js

Для инициализации формы, Вам необходимо вставить даный JS код.

	  /**
	   * Global variables
	   */

	  var $document = $(document),
	    plugins = {
        mailchimp: $('.mailchimp-mailform'),
	      regula: $("[data-constraints]"),  
	      rdInputLabel: $(".form-label"),
	    };

	  /**
	   * Initialize All Scripts
	   */
	  $document.ready(function () {
	    var isNoviBuilder = window.xMode;


	    /**
	     * attachFormValidator
	     * @description  attach form validation to elements
	     */
	    function attachFormValidator(elements) {
	      for (var i = 0; i < elements.length; i++) {
	        var o = $(elements[i]), v;
	        o.addClass("form-control-has-validation").after("<span class='form-validation'></span>");
	        v = o.parent().find(".form-validation");
	        if (v.is(":last-child")) {
	          o.addClass("form-control-last-child");
	        }
	      }

	      elements
	        .on('input change propertychange blur', function (e) {
	          var $this = $(this), results;

	          if (e.type !== "blur") {
	            if (!$this.parent().hasClass("has-error")) {
	              return;
	            }
	          }

	          if ($this.parents('.rd-mailform').hasClass('success')) {
	            return;
	          }

	          if ((results = $this.regula('validate')).length) {
	            for (i = 0; i < results.length; i++) {
	              $this.siblings(".form-validation").text(results[i].message).parent().addClass("has-error")
	            }
	          } else {
	            $this.siblings(".form-validation").text("").parent().removeClass("has-error")
	          }
	        })
	        .regula('bind');

	      var regularConstraintsMessages = [
	        {
	          type: regula.Constraint.Required,
	          newMessage: "The text field is required."
	        },
	        {
	          type: regula.Constraint.Email,
	          newMessage: "The email is not a valid email."
	        },
	        {
	          type: regula.Constraint.Numeric,
	          newMessage: "Only numbers are required"
	        },
	        {
	          type: regula.Constraint.Selected,
	          newMessage: "Please choose an option."
	        }
	      ];


	      for (var i = 0; i < regularConstraintsMessages.length; i++) {
	        var regularConstraint = regularConstraintsMessages[i];

	        regula.override({
	          constraintType: regularConstraint.type,
	          defaultMessage: regularConstraint.newMessage
	        });
	      }
	    }


	    /**
	     * isValidated
	     * @description  check if all elements pass validation
	     */
	    function isValidated(elements, captcha) {
	      var results, errors = 0;

	      if (elements.length) {
	        for (j = 0; j < elements.length; j++) {

	          var $input = $(elements[j]);
	          if ((results = $input.regula('validate')).length) {
	            for (k = 0; k < results.length; k++) {
	              errors++;
	              $input.siblings(".form-validation").text(results[k].message).parent().addClass("has-error");
	            }
	          } else {
	            $input.siblings(".form-validation").text("").parent().removeClass("has-error")
	          }
	        }

	        if (captcha) {
	          if (captcha.length) {
	            return validateReCaptcha(captcha) && errors === 0
	          }
	        }

	        return errors === 0;
	      }
	      return true;
	    }


	    /**
	     * RD Input Label
	     * @description Enables RD Input Label Plugin
	     */

	    if (plugins.rdInputLabel.length) {
	      plugins.rdInputLabel.RDInputLabel();
	    }


	    /**
	     * Regula
	     * @description Enables Regula plugin
	     */

	    if (plugins.regula.length) {
	      attachFormValidator(plugins.regula);
	    }


	    /**
	     * MailChimp Ajax subscription
	     */

	    if (plugins.mailchimp.length) {
	      for (i = 0; i < plugins.mailchimp.length; i++) {
	        var $mailchimpItem = $(plugins.mailchimp[i]),
	          $email = $mailchimpItem.find('input[type="email"]');

	        // Required by MailChimp
	        $mailchimpItem.attr('novalidate', 'true');
	        $email.attr('name', 'EMAIL');

	        $mailchimpItem.on('submit', $.proxy(function (e){
	          e.preventDefault();

	          var $this = this;

	          var data = {},
	            url = $this.attr('action').replace('/post?', '/post-json?').concat('&c=?'),
	            dataArray = $this.serializeArray(),
	            $output = $("#" + $this.attr("data-form-output"));

	          for (i = 0; i < dataArray.length; i++) {
	            data[dataArray[i].name] = dataArray[i].value;
	          }

	          $.ajax({
	            data: data,
	            url: url,
	            dataType: 'jsonp',
	            error: function (resp, text) {
	              $output.html('Server error: ' + text);

	              setTimeout(function () {
	                $output.removeClass("active");
	              }, 4000);
	            },
	            success: function (resp) {
	              $output.html(resp.msg).addClass('active');

	              setTimeout(function () {
	                $output.removeClass("active");
	              }, 6000);
	            },
	            beforeSend: function(data){
	              // Stop request if builder or inputs are invalide
	              if (isNoviBuilder || !isValidated($this.find('[data-constraints]')))
	                return false;

	              $output.html('Submitting...').addClass('active');
	            }
	          });

	          return false;
	        }, $mailchimpItem));
	      }
	    }
	  });


Для подключения формы подписки к своему аккаунту MailChimp, вам необходимо создать список(или выбрать уже существующий) (1) и перейти в "Signup forms" раздел (2).

![](https://www.templatemonster.com/help/quick-start-guide/website-templates/v1-4/img/mailchimp.png)

Выбрать пункт "General forms".

![](https://www.templatemonster.com/help/quick-start-guide/website-templates/v1-4/img/mailchimp-2.png)

Скопировать ссылку из поля "Signup form URL" и перейти по ней.

![](https://www.templatemonster.com/help/quick-start-guide/website-templates/v1-4/img/mailchimp-3.png)

Вас перенаправит на другую страницу, с которой Вам нужно взять url:

![](https://www.templatemonster.com/help/quick-start-guide/website-templates/v1-4/img/mailchimp-4.png)

В данную ссылку необходимо добавить /post после слова subscribe, чтобы ссылка приняла вид

https://********.***.list-manage.com/subscribe/post?u=*******************&id=*********

Данную ссылку необходимо вставить в атрибут action формы подписки, пример:

    <form class="mailchimp-mailform"
    data-form-output="form-output-global"
    action="https://templatemonster.us15.list-manage.com/subscribe/post?u=ba5bb362073a413f48e4a7b90&id=8dc95d9dec"
    method="post">
	    <div class="form-wrap">
	      <label class="form-label rd-input-label" for="inline-email">Enter your e-mail</label>
	      <input class="form-input" id="inline-email" type="email" name="EMAIL" data-constraints="@Email @Required">
	    </div>
	    <button class="button button-primary" type="submit">Subscribe</button>
	  </form>

    <!-- Елемент в который будет выводиться информация об ошибке-->
    <div class="form-output-global"></div>
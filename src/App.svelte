<script>
  import html2canvas from "html2canvas";

  let files, twitterName;
  let tweet = {
    avatar: "",
    name: "",
    username: "",
    content: "",
    stats: {
      retweet: "0",
      quotation: "0",
      like: "0",
    },
    verified: false,
  };

  const convertImgToBase64 = (url, callback, outputFormat) => {
    var canvas = document.createElement("CANVAS");
    var ctx = canvas.getContext("2d");
    var img = new Image();
    img.crossOrigin = "Anonymous";
    img.onload = function () {
      canvas.height = img.height;
      canvas.width = img.width;
      ctx.drawImage(img, 0, 0);
      var dataURL = canvas.toDataURL(outputFormat || "image/png");
      callback.call(this, dataURL);
      // Clean up
      canvas = null;
    };
    img.src = url;
  };

  const tweetFormat = (tweet) => {
    tweet = tweet
      .replace(
        // replace @username
        /@([\w]+)/g,
        "<span class='tag' style='color: rgb(29, 161, 242);'>@$1</span>"
      )
      .replace(
        // replace #hashtag
        /#([\wşçöğüıİ]+)/gi,
        "<span class='hashtag' style='color: rgb(29, 161, 242);'>#$1</span>"
      )
      .replace(
        // replace http://
        /(https?:\/\/[\w\.\/]+)/,
        "<span class='url' style='color: rgb(29, 161, 242);'>$1</span>"
      )
      .replace(/\n/g, "<br />"); // replace new line
    return tweet;
  };

  const formatNumber = (number) => {
    if (!number) {
      // if number is undefined
      number = 0;
    }
    if (number < 1000) {
      return number; // if number is less than 1000
    }
    number /= 1000; // convert to kilobytes
    number = String(number).split("."); // split into integer and decimal parts

    return (
      number[0] + (number[1] > 100 ? "," + number[1].slice(0, 1) + " B" : " B") // add B to the end of the integer part
    );
  };

  const takeSnapShot = function () {
    html2canvas(document.querySelector("#tweet")).then(function (canvas) {
      let link = document.createElement("a"); //create a link
      link.href = canvas.toDataURL("image/png"); //set the link href to the canvas data
      link.download = "tweet.png";
      link.click();
    });
  };

  const handleImage = function (files) {
    const file = files; //files[0];
    const reader = new FileReader(); // create reader
    reader.onload = function (event) {
      // on load callback
      tweet.avatar = event.target.result; // put image data into avatar
    };
    reader.readAsDataURL(file); // read file
  };

  const getUser = function (user) {
    let uri =
      "https://typeahead-js-twitter-api-proxy.herokuapp.com/demo/search?q=";
    fetch(uri + user)
      .then((res) => res.json()) // Transform the data into json
      .then((data) => {
        // Log the data
        let twitter = data[0]; //data[0] is the first user
        console.log(twitter);
        convertImgToBase64(twitter.profile_image_url, function (base64Image) {
          tweet.avatar = base64Image;
        });
        tweet.name = twitter.name; //name
        tweet.username = twitter.screen_name; //username
        tweet.content = twitter.status.text; //tweet content
        tweet.stats.retweet = twitter.status.retweet_count; //retweet count
        tweet.stats.like = twitter.status.favorite_count; //like count
      });
  };
</script>

<div style="display: none;">
  {#if files && files[0]}
    {handleImage(files[0])}
  {/if}
</div>

<main>
  <div class="tweet-settings">
    <div class="title">
      <h1>Tweet Ayarları</h1>
    </div>
    <div class="setting">
      <div class="form-group">
        <label for="name">Ad ve Soyad</label>
        <input
          type="text"
          name="name"
          id="name"
          class="form-control"
          bind:value={tweet.name}
        />
      </div>
      <div class="form-group">
        <label for="username">Kullanıcı Adı</label>
        <input
          type="text"
          id="username"
          class="form-control"
          bind:value={tweet.username}
        />
      </div>
      <div class="form-group">
        <label for="content">Tweet</label>
        <textarea
          name="content"
          id="content"
          class="form-control"
          bind:value={tweet.content}
        />
      </div>
      <div class="form-group">
        <label for="image">Avatar</label>
        <span>
          {#if files && files[0]}
            Seçilen Resim: {files[0].name}
          {:else}
            Profil Resmi seçmek için tıklayın.
          {/if}
        </span>
        <input
          type="file"
          accept="image/*"
          id="image"
          class="form-control"
          bind:files
        />
      </div>
      <div class="form-group">
        <label for="retweet">Retweet</label>
        <input
          type="number"
          min="0"
          id="retweet"
          class="form-control"
          bind:value={tweet.stats.retweet}
        />
      </div>
      <div class="form-group">
        <label for="quotation">Alıntı Tweet</label>
        <input
          type="number"
          min="0"
          id="quotation"
          class="form-control"
          bind:value={tweet.stats.quotation}
        />
      </div>
      <div class="form-group">
        <label for="like">Beğenme</label>
        <input
          type="number"
          min="0"
          id="like"
          class="form-control"
          bind:value={tweet.stats.like}
        />
      </div>
      <div class="form-group">
        <label for="like">Doğrulanmış Hesap</label>
        <select class="form-control" bind:value={tweet.verified}>
          <option value="1">Evet</option>
          <option value="0" selected>Hayır</option>
        </select>
      </div>
      <div class="form-group">
        <button class="button" on:click={takeSnapShot}>Oluştur</button>
      </div>
    </div>
  </div>
  <div class="tweet-container">
    <div class="fetch-info">
      <div class="form-group">
        <input
          type="text"
          placeholder="Twitter kullanıcı adını yazın"
          bind:value={twitterName}
        />
      </div>
      <div class="button-group">
        <button type="button" class="button" on:click={getUser(twitterName)}
          >Bilgileri Çek</button
        >
      </div>
    </div>
    <div class="tweet" id="tweet">
      <div class="tweet-author">
        <div class="tweet-avatar">
          <img
            src={tweet.avatar || "/images/base-avatar.jpg"}
            alt="tweet-avatar"
          />
        </div>
        <div class="tweet-information">
          <div class="tweet-name">
            {tweet.name || "Ad ve Soyad"}
            {#if tweet.verified == 1}
              <span class="icon">
                <svg viewBox="0 0 24 24"
                  ><g
                    ><path
                      d="M22.5 12.5c0-1.58-.875-2.95-2.148-3.6.154-.435.238-.905.238-1.4 0-2.21-1.71-3.998-3.818-3.998-.47 0-.92.084-1.336.25C14.818 2.415 13.51 1.5 12 1.5s-2.816.917-3.437 2.25c-.415-.165-.866-.25-1.336-.25-2.11 0-3.818 1.79-3.818 4 0 .494.083.964.237 1.4-1.272.65-2.147 2.018-2.147 3.6 0 1.495.782 2.798 1.942 3.486-.02.17-.032.34-.032.514 0 2.21 1.708 4 3.818 4 .47 0 .92-.086 1.335-.25.62 1.334 1.926 2.25 3.437 2.25 1.512 0 2.818-.916 3.437-2.25.415.163.865.248 1.336.248 2.11 0 3.818-1.79 3.818-4 0-.174-.012-.344-.033-.513 1.158-.687 1.943-1.99 1.943-3.484zm-6.616-3.334l-4.334 6.5c-.145.217-.382.334-.625.334-.143 0-.288-.04-.416-.126l-.115-.094-2.415-2.415c-.293-.293-.293-.768 0-1.06s.768-.294 1.06 0l1.77 1.767 3.825-5.74c.23-.345.696-.436 1.04-.207.346.23.44.696.21 1.04z"
                    /></g
                  ></svg
                >
              </span>
            {/if}
          </div>
          <div class="tweet-username">@{tweet.username || "kullaniciadi"}</div>
        </div>
      </div>
      <div class="tweet-content">
        <p>
          {@html tweetFormat(tweet.content) ||
            "Bu alana tweet içeriği gelecektir."}
        </p>
      </div>
      <div class="tweet-stats">
        <div class="retweet">
          <strong> {formatNumber(tweet.stats.retweet) || "0"} </strong>
          <span>Retweet</span>
        </div>
        <div class="quotation">
          <strong> {formatNumber(tweet.stats.quotation) || "0"} </strong>
          <span>Alıntı Tweetler</span>
        </div>
        <div class="like">
          <strong> {formatNumber(tweet.stats.like) || "0"} </strong>
          <span>Beğeni</span>
        </div>
      </div>
      <div class="tweet-actions">
        <div class="reply">
          <span class="icon">
            <svg viewBox="0 0 24 24" aria-hidden="true"
              ><g
                ><path
                  d="M14.046 2.242l-4.148-.01h-.002c-4.374 0-7.8 3.427-7.8 7.802 0 4.098 3.186 7.206 7.465 7.37v3.828c0 .108.044.286.12.403.142.225.384.347.632.347.138 0 .277-.038.402-.118.264-.168 6.473-4.14 8.088-5.506 1.902-1.61 3.04-3.97 3.043-6.312v-.017c-.006-4.367-3.43-7.787-7.8-7.788zm3.787 12.972c-1.134.96-4.862 3.405-6.772 4.643V16.67c0-.414-.335-.75-.75-.75h-.396c-3.66 0-6.318-2.476-6.318-5.886 0-3.534 2.768-6.302 6.3-6.302l4.147.01h.002c3.532 0 6.3 2.766 6.302 6.296-.003 1.91-.942 3.844-2.514 5.176z"
                /></g
              ></svg
            >
          </span>
        </div>
        <div class="retweet" on:click={() => tweet.stats.retweet++}>
          <span class="icon">
            <svg viewBox="0 0 24 24" aria-hidden="true"
              ><g
                ><path
                  d="M23.77 15.67c-.292-.293-.767-.293-1.06 0l-2.22 2.22V7.65c0-2.068-1.683-3.75-3.75-3.75h-5.85c-.414 0-.75.336-.75.75s.336.75.75.75h5.85c1.24 0 2.25 1.01 2.25 2.25v10.24l-2.22-2.22c-.293-.293-.768-.293-1.06 0s-.294.768 0 1.06l3.5 3.5c.145.147.337.22.53.22s.383-.072.53-.22l3.5-3.5c.294-.292.294-.767 0-1.06zm-10.66 3.28H7.26c-1.24 0-2.25-1.01-2.25-2.25V6.46l2.22 2.22c.148.147.34.22.532.22s.384-.073.53-.22c.293-.293.293-.768 0-1.06l-3.5-3.5c-.293-.294-.768-.294-1.06 0l-3.5 3.5c-.294.292-.294.767 0 1.06s.767.293 1.06 0l2.22-2.22V16.7c0 2.068 1.683 3.75 3.75 3.75h5.85c.414 0 .75-.336.75-.75s-.337-.75-.75-.75z"
                /></g
              ></svg
            >
          </span>
        </div>
        <div class="like" on:click={() => tweet.stats.like++}>
          <span class="icon">
            <svg viewBox="0 0 24 24" aria-hidden="true"
              ><g
                ><path
                  d="M12 21.638h-.014C9.403 21.59 1.95 14.856 1.95 8.478c0-3.064 2.525-5.754 5.403-5.754 2.29 0 3.83 1.58 4.646 2.73.814-1.148 2.354-2.73 4.645-2.73 2.88 0 5.404 2.69 5.404 5.755 0 6.376-7.454 13.11-10.037 13.157H12zM7.354 4.225c-2.08 0-3.903 1.988-3.903 4.255 0 5.74 7.034 11.596 8.55 11.658 1.518-.062 8.55-5.917 8.55-11.658 0-2.267-1.823-4.255-3.903-4.255-2.528 0-3.94 2.936-3.952 2.965-.23.562-1.156.562-1.387 0-.014-.03-1.425-2.965-3.954-2.965z"
                /></g
              ></svg
            >
          </span>
        </div>
        <div class="share">
          <span class="icon">
            <svg viewBox="0 0 24 24" aria-hidden="true"
              ><g
                ><path
                  d="M17.53 7.47l-5-5c-.293-.293-.768-.293-1.06 0l-5 5c-.294.293-.294.768 0 1.06s.767.294 1.06 0l3.72-3.72V15c0 .414.336.75.75.75s.75-.336.75-.75V4.81l3.72 3.72c.146.147.338.22.53.22s.384-.072.53-.22c.293-.293.293-.767 0-1.06z"
                /><path
                  d="M19.708 21.944H4.292C3.028 21.944 2 20.916 2 19.652V14c0-.414.336-.75.75-.75s.75.336.75.75v5.652c0 .437.355.792.792.792h15.416c.437 0 .792-.355.792-.792V14c0-.414.336-.75.75-.75s.75.336.75.75v5.652c0 1.264-1.028 2.292-2.292 2.292z"
                /></g
              ></svg
            >
          </span>
        </div>
      </div>
    </div>
  </div>
</main>

<style lang="scss">
  main {
    display: flex;
    .tweet-settings {
      width: 350px;
      min-height: 100%;
      min-width: 350px;
      padding: 1rem;
      border-right: 1px solid rgb(47, 51, 54);
      background-color: rgb(0, 0, 0);
      overflow: auto;
      height: 100%;
      min-height: 100vh;
      .title {
        h1 {
          font-size: 24px;
          font-weight: 400;
          border-bottom: 1px solid rgb(47, 51, 54);
          padding-bottom: 20px;
          margin-bottom: 20px;
        }
      }
      .setting {
        height: 100%;
        .form-group {
          width: 100%;
          display: flex;
          flex-direction: column;
          position: relative;
          span {
            padding: 10px 0 15px;
            display: block;
            border-bottom: 1px solid rgb(47, 51, 54);
          }
          &:not(:last-child) {
            margin-bottom: 1rem;
          }
          label {
            font-size: 1rem;
            margin-bottom: 5px;
            display: block;
            cursor: pointer;
            color: rgba(255, 255, 255, 0.65);
          }
          .form-control {
            appearance: none;
            resize: none;
            width: 100%;
            padding: 10px 0 15px;
            border: none;
            background-color: #000;
            border-bottom: 1px solid rgb(47, 51, 54);
            color: rgb(255, 255, 255);
            font-size: 1rem;
          }
          .button {
            width: 100%;
            height: 45px;
            border-radius: 4px;
            font-size: 16px;
            color: #fff;
            background: rgb(29, 161, 242);
            cursor: pointer;
            transition: 250ms all;
            &:hover {
              background-color: darken($color: #1da1f2, $amount: 7.5%);
            }
            &:active {
              background-color: darken($color: #1da1f2, $amount: 12.5%);
            }
          }
          input[type="file"] {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            opacity: 0;
          }
        }
      }
    }
    .tweet-container {
      flex: 1;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      .fetch-info {
        width: 600px;
        display: flex;
        gap: 1rem;
        .form-group {
          flex: 1;
          input {
            width: 100%;
            background: transparent;
            border: 1px solid #2f3336;
            color: #fff;
            padding: 0.725rem;
            font-size: 1rem;
            border-radius: 4px;
          }
        }
        .button-group {
          .button {
            width: 100%;
            border-radius: 4px;
            font-size: 16px;
            color: #fff;
            background: #1da1f2;
            cursor: pointer;
            transition: 250ms all;
            padding: 0.725rem;
            margin-top: 1px;
            &:hover {
              background-color: darken($color: #1da1f2, $amount: 7.5%);
            }
            &:active {
              background-color: darken($color: #1da1f2, $amount: 12.5%);
            }
          }
        }
      }
      .tweet {
        border: 1px solid #2f3336;
        width: 600px;
        margin: 25px auto;
        padding: 0 16px;
        background: #000;
        .tweet-author {
          height: 48px;
          margin-top: 12px;
          display: flex;
          align-items: center;
          margin-bottom: 4px;
          .tweet-avatar {
            width: 48px;
            height: 48px;
            margin-right: 16px;
            img {
              width: 100%;
              height: 100%;
              border-radius: 50%;
              object-fit: cover;
              cursor: auto;
            }
          }
          .tweet-information {
            .tweet-name {
              color: #fff;
              font-weight: 700;
              display: flex;
              align-items: center;
              height: 19px;
              display: flex;
              align-items: center;
              gap: 2.5px;
              span {
                width: 19px;
                height: 19px;
              }
              svg {
                fill: #fff;
                width: 100%;
                height: 100%;
              }
            }
            .tweet-username {
              color: #6e767d;
            }
          }
        }
        .tweet-content {
          padding: 12px 0;
          p {
            font-size: 23px;
            line-height: 28px;
            color: #fff;
            overflow: hidden;
          }
        }
        .tweet-stats {
          height: 53px;
          display: flex;
          align-items: center;
          font-size: 15px;
          color: #6e767d;
          border-top: 1px solid #2f3336;
          gap: 20px;
          strong {
            color: #fff;
          }
        }
        .tweet-actions {
          height: 48px;
          display: flex;
          border-top: 1px solid #2f3336;
          color: #fff;
          align-items: center;
          justify-content: space-around;
          > div {
            border-radius: 50%;
            transition: 250ms all;
          }
          .reply {
            &:hover {
              background-color: rgba(29, 155, 240, 0.1);
              svg {
                fill: rgb(29, 155, 240);
              }
            }
          }
          .retweet {
            &:hover {
              background-color: rgba(0, 186, 124, 0.1);
              svg {
                fill: rgb(0, 186, 124);
              }
            }
          }
          .like {
            &:hover {
              background-color: rgba(249, 24, 128, 0.1);
              svg {
                fill: rgb(249, 24, 128);
              }
            }
          }
          .share {
            &:hover {
              background-color: rgba(29, 156, 240, 0.1);
              svg {
                fill: rgb(29, 155, 240);
              }
            }
          }
          span.icon {
            cursor: pointer;
            width: 38px;
            height: 38px;
            display: flex;
            align-items: center;
            justify-content: center;
            border-radius: 50%;
            svg {
              width: 24px;
              height: 24px;
              fill: #6e767d;
            }
          }
        }
      }
    }
  }
</style>

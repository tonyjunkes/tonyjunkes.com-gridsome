<template>
  <Layout>
    <div class="hero container-inner mx-auto flex flex-col sm:flex-row justify-between py-16">
      <div class="text-4xl font-bold w-full sm:w-3/5 text-center sm:text-left">
        <div id="about" class="leading-tight">Hey there, <span class="text-blue-700">I'm Tony Junkes.</span></div>
      </div>
    </div> <!-- end hero -->

    <div class="overflow-x-hidden border-gray-200 border-b">
      <div class="get-to-know-me container-inner mx-auto text-xl pb-16 relative">
        <div class="flex flex-col sm:flex-row justify-between items-center mb-16">
          <div><g-image src="../../static/avatar.jpg" alt="avatar" class="w-32 h-32 rounded-full mb-8 lg:mb-0" /></div>
          <div class="flex-1 text-lg sm:text-xl ml-6">
            I’m a Software Engineer living in South Florida and have been working in web development for over a decade; primarily using CFML, Java &amp; JavaScript. I enjoy learning new things and that's the primary reason I started writing about my experiences and sharing them here. You can get a glimpse of what I've been working on / tinkering with over on <a href="https://github.com/tonyjunkes" target="_blank">GitHub</a>.
          </div>
        </div>
        <!-- <div class="responsive-container">
          <iframe
            src="https://player.twitch.tv/?channel=tonycreatestv&parent=localhost&parent=tonyjunkes.com&muted=true"
            height="480"
            width="640"
            frameborder="0"
            scrolling="no"
            allowfullscreen="true">
          </iframe>
        </div> -->
      </div> <!-- end get-to-know me -->
    </div>

    <div class="overflow-x-hidden">
      <div class="contact-me bg-background-secondary pt-16">
        <div class="container-inner mx-auto text-xl pb-4 relative">
          <h2 class="font-bold mb-6" id="contact">Questions? Comments? Hit me up!</h2>

          <div class="text-lg sm:text-lg mb-16">
            <form class="mb-12" action="https://formspree.io/mvorzaoe" method="post">
              <input type="hidden" name="_subject" value="tonyjunkes.com - Contact Form Submission">
              <input type="text" name="_gotcha" style="display:none">
              <div class="flex flex-wrap mb-6 -mx-4">
                  <div class="w-full md:w-1/2 mb-6 md:mb-0 px-4">
                      <label class="block mb-2 text-copy-primary" for="name">
                          Name
                      </label>

                      <input type="text" name="name" id="name" placeholder="Jon Irenicus" class="block w-full bg-background-form border border-border-color-primary shadow rounded outline-none focus:border-blue-700 mb-2 p-4" required>
                  </div>

                  <div class="w-full px-4 md:w-1/2">
                      <label class="block text-copy-primary mb-2" for="email">
                          Email Address
                      </label>

                      <input type="email" name="_replyto" id="email" placeholder="email@example.com"  class="block w-full bg-background-form border border-border-color-primary shadow rounded outline-none focus:border-blue-700 mb-2 p-4" required>
                  </div>
              </div>

              <div class="w-full mb-12">
                  <label class="block text-copy-primary mb-2" for="message">
                      Message
                  </label>

                  <textarea id="message" rows="5" name="message" class="block w-full bg-background-form border border-border-color-primary shadow rounded outline-none appearance-none focus:border-blue-700 mb-2 px-4 py-4" placeholder="Enter your message here." required></textarea>
              </div>

              <div class="flex justify-end w-full">
                  <input type="submit" value="Submit" class="block bg-blue-700 hover:bg-blue-800 text-white text-sm font-semibold tracking-wide uppercase shadow rounded cursor-pointer px-6 py-3">
              </div>
          </form>
          </div>
        </div>
      </div> <!-- end contact-me -->
    </div>

  </Layout>
</template>

<script>
export default {
  metaInfo: {
    title: 'Home'
  }
}
</script>


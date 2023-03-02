# Confidence Coin Pitch Deck

{% include header.md %}

<meta name="viewport" content="width=device-width, initial-scale=1.0">
<link rel="stylesheet" type="text/css" href="https://cdn.jsdelivr.net/npm/slick-carousel/slick/slick.css"/>
<link rel="stylesheet" type="text/css" href="https://cdn.jsdelivr.net/npm/slick-carousel/slick/slick-theme.css"/>
<style>
  .carousel iframe {
    max-width: 600px;
    width: 100%;
    height: 480px;
    border: none;
  }
  .button-container {
    text-align: center;
    left: 50%;
    opacity: 10%;
  }
  button {
    background-color: #008CBA; /* Blue background color */
    border: none; /* Remove default button border */
    color: white; /* White text color */
    padding: 12px 24px; /* Add some padding to the button */
    text-align: center; /* Center the button text */
    text-decoration: none; /* Remove default underline */
    display: inline-block; /* Set button display to inline-block */
    font-size: 16px; /* Set font size */
    margin: 4px 2px; /* Add margin around the button */
    cursor: pointer; /* Add cursor pointer */
    border-radius: 4px; /* Add border radius */
    width: 250px
  }

  button:hover {
  background-color: #00688B; /* Darker blue background color on hover */
  }
  .slick-initialized {
    display: block !important;
  }
  .slick-slide {
    text-align: center;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .slick-dots {
    width: 100%;
    text-align: center;
    margin: 0;
    padding: 0;
    z-index: 10;
    list-style: none;
  }
  
</style>

  <div class="carousel" style="display: none;">
    <div><iframe src="Title"></iframe></div>
    <div><iframe src="Problem"></iframe></div>
    <div><iframe src="Solution"></iframe></div>
    <div><iframe src="Market"></iframe></div>
    <div><iframe src="Competitive-Advantage"></iframe></div>
    <div><iframe src="Business-Model"></iframe></div>
    <div><iframe src="Team"></iframe></div>
    <div><iframe src="Financial-Projections"></iframe></div>
    <div><iframe src="Conclusion"></iframe></div>
  </div>

  <div class="button-container">
    <button class="prev">Previous</button>
    <button class="next">Next</button>
  </div>

  <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/slick-carousel/slick/slick.min.js"></script>
  <script>
    $('.carousel').slick({
      arrows: false,
      dots: true,
      adaptiveHeight: true
    });
    
    $('.prev').click(function(){
      $('.carousel').slick('slickPrev');
    });
    
    $('.next').click(function(){
      $('.carousel').slick('slickNext');
    });
    
    $('.carousel').show();
    $('.button-container').css('opacity', '1')
  </script>

# Contact us
- [LinkedIn](https://www.linkedin.com/in/ilyagazman/)
- <a href='mai&#108;t&#111;&#58;g&#37;&#54;1z&#109;&#97;%6E19%&#51;8%3&#54;&#64;gma&#105;l&#46;co%6D'>&#103;a&#122;man198&#54;&#64;gm&#97;il&#46;co&#109;</a>


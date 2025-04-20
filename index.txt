import "./style.css";

const video = document.getElementById("video");
const canvas = document.getElementById("canvas");
const captureButton = document.getElementById("capture-btn");
const imagesContainer = document.getElementById("images-container");

// Create and insert the "Download All as Strip" button
const downloadAllButton = document.createElement("button");
downloadAllButton.id = "download-all-btn";
downloadAllButton.textContent = "Download All as Strip";
downloadAllButton.style.marginTop = "15px";
downloadAllButton.style.padding = "10px 15px";
downloadAllButton.style.fontSize = "14px";
imagesContainer.before(downloadAllButton);

const capturedImagesData = []; // Store image data

// Initialize camera when the page loads
window.addEventListener("DOMContentLoaded", initCamera);

// Capture image when the button is clicked
captureButton.addEventListener("click", captureImage);

// Download all images as a vertical strip
downloadAllButton.addEventListener("click", downloadStrip);

// Get access to the camera
async function initCamera() {
  try {
    const stream = await navigator.mediaDevices.getUserMedia({ video: true });
    video.srcObject = stream;
  } catch (err) {
    console.error("Error accessing the camera: ", err);
  }
}

// Capture a still image
function captureImage() {
  const context = canvas.getContext("2d");

  context.drawImage(video, 0, 0, canvas.width, canvas.height);

  const imgData = canvas.toDataURL("image/png");
  capturedImagesData.push(imgData);

  const capturedImage = new Image();
  capturedImage.src = imgData;
  capturedImage.width = 200;
  capturedImage.height = 150;

  const downloadLink = document.createElement("a");
  downloadLink.href = imgData;
  downloadLink.download = `captured-${Date.now()}.png`;
  downloadLink.textContent = "Download Photo";
  downloadLink.style.display = "block";
  downloadLink.style.marginTop = "5px";

  const container = document.createElement("div");
  container.style.marginBottom = "10px";
  container.style.border = "1px solid #ddd";
  container.style.padding = "8px";
  container.style.borderRadius = "8px";
  container.style.backgroundColor = "#f0f0f0";

  container.appendChild(capturedImage);
  container.appendChild(downloadLink);

  imagesContainer.prepend(container);
}

// Combine all captured images into a vertical strip and download as one image
function downloadStrip() {
  if (capturedImagesData.length === 0) return;

  const imgWidth = 200;
  const imgHeight = 150;
  const stripCanvas = document.createElement("canvas");
  stripCanvas.width = imgWidth;
  stripCanvas.height = imgHeight * capturedImagesData.length;
  const ctx = stripCanvas.getContext("2d");

  let imagesLoaded = 0;

  capturedImagesData.forEach((dataURL, index) => {
    const img = new Image();
    img.src = dataURL;
    img.onload = () => {
      ctx.drawImage(img, 0, imgHeight * index, imgWidth, imgHeight);
      imagesLoaded++;

      // When all images are drawn, trigger download
      if (imagesLoaded === capturedImagesData.length) {
        const stripData = stripCanvas.toDataURL("image/png");
        const downloadLink = document.createElement("a");
        downloadLink.href = stripData;
        downloadLink.download = `photo_strip_${Date.now()}.png`;
        downloadLink.click();
      }
    };
  });
}

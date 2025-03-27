import React, { useState } from "react";
import { Button } from "./components/ui/button";
import { Input } from "./components/ui/input";
import { Card, CardContent } from "./components/ui/card";
import { UploadCloud } from "lucide-react";
import * as pdfjsLib from "pdfjs-dist";
import mammoth from "mammoth";

const resumeTemplates = {
  USA: "USA_Resume_Template.pdf",
  UK: "UK_Resume_Template.pdf",
  Canada: "Canada_Resume_Template.pdf",
  Australia: "Australia_Resume_Template.pdf",
  Germany: "Germany_CV_Template.pdf",
};

function ATSResumeScanner() {
  const [uploadedFile, setUploadedFile] = useState(null);
  const [analysisResult, setAnalysisResult] = useState(null);
  const [selectedCountry, setSelectedCountry] = useState("USA");

  const handleFileUpload = (event) => {
    const file = event.target.files[0];
    if (file) {
      setUploadedFile(file);
      analyzeResume(file);
    }
  };

  const analyzeResume = async (file) => {
    const keywords = ["experience", "skills", "education", "certifications", "projects"];
    const fileText = await extractTextFromFile(file);
    if (typeof fileText !== "string") {
      setAnalysisResult("Error processing file. Please try again.");
      return;
    }
    const keywordMatches = keywords.filter((keyword) => fileText.toLowerCase().includes(keyword)).length;
    const score = (keywordMatches / keywords.length) * 100;
    
    setAnalysisResult(`Your resume is ${Math.round(score)}% ATS-friendly. Consider optimizing keywords and formatting.`);
  };

  const extractTextFromFile = async (file) => {
    try {
      if (file.type === "application/pdf") {
        return await extractTextFromPDF(file);
      } else if (file.type === "application/vnd.openxmlformats-officedocument.wordprocessingml.document") {
        return await extractTextFromDOCX(file);
      } else {
        return "Unsupported file format. Please upload a PDF or DOCX.";
      }
    } catch (error) {
      return `Error extracting text: ${error.message}`;
    }
  };

  const extractTextFromPDF = async (file) => {
    const reader = new FileReader();
    return new Promise((resolve) => {
      reader.onload = async function () {
        try {
          const typedArray = new Uint8Array(reader.result);
          const pdf = await pdfjsLib.getDocument(typedArray).promise;
          let text = "";
          for (let i = 1; i <= pdf.numPages; i++) {
            const page = await pdf.getPage(i);
            const content = await page.getTextContent();
            text += content.items.map((item) => item.str).join(" ") + " ";
          }
          resolve(text);
        } catch (error) {
          resolve(`Error extracting PDF text: ${error.message}`);
        }
      };
      reader.readAsArrayBuffer(file);
    });
  };

  const extractTextFromDOCX = async (file) => {
    const reader = new FileReader();
    return new Promise((resolve) => {
      reader.onload = async function () {
        try {
          const result = await mammoth.extractRawText({ arrayBuffer: reader.result });
          resolve(result.value);
        } catch (error) {
          resolve(`Error extracting DOCX text: ${error.message}`);
        }
      };
      reader.readAsArrayBuffer(file);
    });
  };

  return (
    <div className="flex flex-col items-center p-6 space-y-6">
      <h1 className="text-2xl font-bold">ATS Resume Scanner & Optimizer</h1>
      <Card className="w-full max-w-md p-4 text-center">
        <CardContent>
          <label htmlFor="resumeUpload" className="cursor-pointer flex flex-col items-center border-2 border-dashed p-6 rounded-lg hover:bg-gray-100">
            <UploadCloud size={40} className="text-gray-500" />
            <p className="mt-2">Upload your resume (PDF or DOCX)</p>
            <Input id="resumeUpload" type="file" accept=".pdf,.docx" className="hidden" onChange={handleFileUpload} />
          </label>
        </CardContent>
      </Card>
      <div className="w-full max-w-md">
        <label className="block text-sm font-medium">Select Country Format</label>
        <select className="w-full p-2 border rounded-lg" value={selectedCountry} onChange={(e) => setSelectedCountry(e.target.value)}>
          {Object.keys(resumeTemplates).map((country) => (
            <option key={country} value={country}>{country}</option>
          ))}
        </select>
      </div>
      <Button className="mt-4" onClick={() => window.open(resumeTemplates[selectedCountry], "_blank")}>Download {selectedCountry} Resume Template</Button>
      {uploadedFile && <p className="text-green-600">File Uploaded: {uploadedFile.name}</p>}
      {analysisResult && (
        <Card className="w-full max-w-md p-4 text-center border border-gray-300">
          <CardContent>
            <p className="text-lg font-medium">Analysis Result</p>
            <p className="mt-2 text-gray-600">{analysisResult}</p>
          </CardContent>
        </Card>
      )}
      <Button className="mt-4">Download Optimized Resume</Button>
    </div>
  );
}

export default ATSResumeScanner;
